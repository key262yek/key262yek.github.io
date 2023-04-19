---
title: "Concept of Diffusion model"
date: 2023-04-19
categories:
  - Programming
tags:
  - Langevin equation
  - Generative Modeling
  - Diffusion model
published: true
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

Diffusion model은 Sohl-Dickstein, J.의 2015년 논문[^1]과 Y. Song의 2019년 논문[^2]을 시작으로 많은 영역에서 SOTA를 갱신하고 있는 근래에 가장 핫한 Generative model입니다.
두 논문에서는 Image를 생성했지만, 이후 더 다양한 데이터 형태의 Diffusion으로 확대해 나가면서 3D image[^3], audio[^4], video[^5], Protein[^6][^7], Molecule[^8] 나아가 (무한 차원인) 함수[^9]을 생성하는데 쓰이고 있습니다. 
이번 글에서는 이렇게 핫한 Diffusion model의 일반적인 구조를 설명해보고자 합니다. 

[^1]: Sohl-Dickstein, J. et al. "Deep Unsupervised Learning using Nonequilibrium Thermodynamics" *PMLR* (2015)

[^2]: Yang Song, and Stefano Ermon, "Generative Modeling by Estimating Gradients of the Data Distribution", *NeurIPS* (2019)

[^3]: Ben Poole, Ajay Jain, Jonathan T Barron, and Ben Mildenhall. "Dreamfusion: Text-to-3d using 2d diffusion". *arXiv* (2022)

[^4]: Zhifeng Kong, Wei Ping, Jiaji Huang, Kexin Zhao, and Bryan Catanzaro. "Diffwave: A versatile diffusion model for audio synthesis". *arXiv* (2020)

[^5]: Vikram Voleti, Alexia Jolicoeur-Martineau, and Christopher Pal. "Mcvd: Masked conditional video diffusion for prediction, generation, and interpolation". *NeurIPS* (2022)

[^6]: Kevin E Wu, Kevin K Yang, Rianne van den Berg, James Y Zou, Alex X Lu, and Ava P Amini. "Protein structure generation via folding diffusion". *arXiv* (2022)

[^7]: Joseph L. Watson et al. "Broadly applicable and accurate protein design by integrating structure prediction networks and diffusion generative models". *bioRXiv* (2022)

[^8]: Minkai Xu, Lantao Yu, Yang Song, Chence Shi, Stefano Ermon, and Jian Tang. "Geodiff: A geometric diffusion model for molecular conformation generation". *arXiv* (2022)

[^9]: Jae Hyun Lim et al. "Score-based diffusion models in function space". *arXiv* (2023)


## Diffusion model의 기본 구조
Diffusion model은 간단히 말하면 '`Data distribution`의 `reverse diffusion`을 연산해 random initialization에서 synthetic data를 `만들어내는 모델`(Generative model)'이라 할 수 있습니다. 각각의 키워드에 대해서 자세히 살펴봅시다. 

### Data distribution and Generative model
우리가 A를 만들어내고 싶다면 먼저 명확히 해야하는 것이 크게 두 가지 있습니다.
첫번째는 'A는 어떤 공간의 원소인지'에 대한 답을 알아야합니다.
예를 들어, 이미지는 어떤 공간에 속할까요? 
디지털 관점에서 이미지란 각 pixel의 RGB 정보를 담은 벡터입니다.
따라서 $w \times h$ 크기 이미지는 $\{0, 1, \cdots, 255\}^{3wh}$의 원소일 것입니다.
RGB 정보를 255로 나누어 $[0, 1]$ 사이 실수로 맵핑한다면 이미지는 $[0, 1]^{3wh}$의 원소로도 볼 수 있습니다.
Audio는 time step $t = 1, \cdots, T$에서의 소리신호 $a_t$의 형태로 이뤄져있으니 $\mathbb{R}_+^{T}$의 원소이고,
Molecule은 $N$개 종류의 원자를 node로 하고, 결합을 edge로 하는 임의의 사이즈를 가진 네트워크로 볼 수 있으므로 $\cup_{n=1}^\infty \qty{ 1, \cdots, N }^n \qty{ 0, 1, 2, 3 \}^{n(n - 1)/2}$의 원소가 됩니다.

그렇다면 공간 $[0, 1]^{3wh}$의 모든 벡터는 이미지라고 할 수 있을까요?
$\mathbb{R}_+^{T}$의 모든 원소는 소리일까요?
$\cup_{n=1}^\infty \qty{ 1, \cdots, N }^n \qty{ 0, 1, 2, 3 \}^{n(n - 1)/2}$의 원소는 항상 어떤 molecule에 대응된다고 할 수 있을까요?
그렇지 않을겁니다.
단적으로 아래의 왼쪽 그림을 두고 우리는 이미지라고 하지 않습니다.

Random noise              |  Real Image
:-------------------------:|:-------------------------:
![noise_image](https://key262yek.github.io/assets/images/noise_image.png)  |  ![real_image](https://key262yek.github.io/assets/images/real_image_cifar-10.jpg)

이것은 두 번째 질문 '공간 속 어떤 원소를 A라 부를 것인가'와 관련있습니다.
만약 A가 만족해야 하는 `조건`이 있고 조건을 만족하는 모든 데이터를 A라 부를 수 있다면, 
우리는 조건을 만족하는 임의의 데이터를 만들어냄으로써 A를 생성할 수 있을 것입니다.

하지만 만약 조건을 명확히 하기 어려운 상황이라면 어떻게 해야할까요?
'그림' 혹은 '사진'이 만족해야하는 조건은 무엇일까요?
당장에 제시하기 어렵다면, 적절한 조건을 찾아내는 방법에는 어떤 것이 있을까요?

기계학습은 조건을 학습 데이터로부터 기계 스스로 찾도록 유도하는데, 
우리는 이를 두고 '기계가 `데이터 분포`를 학습한다'고 이야기합니다.
쉽게 풀어 설명하자면 `A로 판단된 데이터`가 전체 공간 중 어디에 위치하는지를 학습하고, 나중에 데이터가 A일 확률이 높은 위치에 있다면 A로 보는 방식을 쓰겠단거죠.

여기에는 한 가지 중요한 가정이 들어가는데 `A의 학습 데이터와 비슷하면 A다`는 가정입니다.
예를 들어, 우리가 이미지라고 인식하는 데이터에서 픽셀 몇 개의 값을 조금 바꾸거나, 전체에 매우 작은 noise를 끼우더라도 우리는 그 결과도 이미지라고 인식합니다. 이 점에 착안해 학습 데이터가 몰려있는 위치 주변의 데이터, 즉 학습 데이터와 가까운 데이터를 만들어내도록 학습하면 우리는 꽤 그럴듯한 가짜 데이터를 만들어낼 수 있게 되는거죠.
(만약 데이터가 조금만 바껴도 판단이 쉽게 바뀌는 경우에는 데이터 군의 영역이 작아지고 경계가 많아지므로, 데이터 군 사이의 경계를 학습하는 모델의 학습 데이터가 그만큼 더 많이 필요하게 됩니다)

다만 이 방법은 학습 데이터의 분포에 크게 영향을 받기 때문에, 원하는 데이터의 모분포를 충분히 모사할 수 있도록 bias가 없는 학습 데이터를 구성하는 것이 중요합니다.

이런 관점에서 Generative model은 큰 틀에서 'random number generator'입니다.
앞서 학습한 data distribution을 따르는 임의의 숫자를 만들어내는 모델인 것이죠.

### Reverse diffusion
Diffusion model이 random data를 만들어내는 방법의 핵심은 reverse diffusion입니다. 
역확산이란 이름에서 알 수 있듯이, 확산의 과정을 역행한다는 것인데 이게 무슨 의미일까요?
역행하기 위해서 먼저 데이터의 확산이 무엇인가를 먼저 생각해봅시다.

물리적으로 확산이란 밀도가 높은 곳에서 낮은 곳으로 일어나는 입자들의 흐름을 의미합니다.
이를 데이터의 관점에 대응시키기 위해, 앞서 설명했던 '데이터의 공간'을 상상하고 그 안에 수많은 데이터들이 각자 점으로 표시되어 있다고 가정해봅시다. 
그리고 그 데이터에 매우 작은 noise를 주어 데이터 공간 속에서 미세하게 움직이도록 해봅시다.
이를 반복한다면 데이터는 데이터 공간 속에서 점차 확산되어 퍼질 것이고, 결국에는 모든 공간을 균등한 비율로 덮게 될 것입니다.

**Original Data**
{% include plots/real_data.html %}  

**After diffusion**
{% include plots/noisy_data.html %}

즉, 확산의 과정을 거치면서 각 데이터는 데이터 공간 속 임의의 점과 (확률적으로) 대응되게 됩니다.
따라서 우리가 확산을 역행할 수 있다면, 데이터 공간 속 임의의 점에서 (확률적으로) 유효한 데이터를 만들어낼 수 있음을 의미합니다.
Reverse diffusion의 방법은 모델마다 조금씩 다른데 다음 section에서 보다 자세히 알아보도록 합시다. 


## Denoising mechanism

### Reverse diffusion kernel 

### Langevin Monte Carlo method

## Pros and Cons, Limitation

## References
