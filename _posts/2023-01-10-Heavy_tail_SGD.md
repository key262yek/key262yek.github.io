---
title: "Better performance of SGD than as we expected"
date: 2023-01-10
categories:
  - Programming
tags:
  - Deep Learning
  - Levy walk
published: true
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

해당 포스트는 `Umut Simsekli`를 비롯한 TELECOM-PARIS의 연구자들의 Review [`On the Heavy-Tailed Theory of Stochastic Gradient Descent for Deep
Neural Networks`](https://arxiv.org/pdf/1912.00018.pdf)를 정리한 글입니다. 

(Forward-Forward algorithm을 활용하는 경우를 제외하고) 현재의 Deep Learning 방법론에서 모델 학습에는 Loss function을 최소화 하기 위해서 Gradient descent 방법을 활용합니다. 
그 중에서 Stochastic Gradient Descent(SGD)는 매번 모든 데이터를 이용해 모델 가중치를 새로 계산하는 것이 아니라, 
무작위로 뽑힌 일부의 데이터의 loss를 최소화하도록 하는 방법을 따릅니다. 
모든 데이터를 한 번에 이용하는 경우에는 Gradient의 방향이 결정적이지만, 
데이터를 무작위로 뽑아낸다는 측면에서 SGD 과정을 통해 학습하는 모델의 가중치 행렬의 움직임은 `Random walk`이라 할 수 있습니다. 

해당 논문에서는 SGD 방법에 따라 학습되는 모델의 움직임에 대한 보다 엄밀한 분석을 제시합니다.
기존 연구들에서는 SGD의 효과를 분석하기 위해서 모델의 Random walk을 임의로 `Brownian motion`(다른 이름으로 `Gaussian noise`)으로 가정하곤 했습니다.
이러한 가정은 수학적으로 모델의 운동을 분석하는 확률미분방정식을 간소화할 수 있다는 이점이 있고,
또한 Brownian particle의 움직임에 대한 기존 연구 역시 다른 운동 양상에 비해 월등히 많아 이에 빗대어 Deep Learning 모델의 학습을 이해하기 수월했을 것입니다. 

하지만 만약 모델의 움직임이 Brownian motion의 성격과 매우 다르다면, 그에 기대어 행해온 대부분의 연구들은 그 신뢰도를 의심받을 수밖에 없습니다. 
이 논문은 SGD를 따르는 모델의 학습 양상이 Brownian motion이 아니라 Levy walk을 따른다는 점을 보이고, 그에 따라 달라지는 여러 분석 결과들을 나열하고 있습니다. 

![Levy_vs_Brownian](https://key262yek.github.io/assets/images/Motion-path-in-Levy-flight-and-Brownian-random-walk.png)

Brownian motion을 가정했을 때의 여러가지 결과
- Learing rate / Batch size 가 최종 minima의 width를 결정한다. (Width가 넓어야 성능이 좋다)
- 해당 minima에 도달하기까지 걸리는 시간이 dimension 증가에 따라 exponentially 커진다. 
  - minima에 도달하는데까지 걸리는 시간은 polynomial time이지만
  - minima에서 탈출해 다른 minima로 가는데 걸리는 시간이 exponentially 증가하기 때문에 그렇다. 
- Minima의 깊이가 깊어짐에 따라 탈출 시간이 exponentially 증가한다. 
- Minima의 너비에 대해서는 Polynomial time으로 증가함. 
  
Empirical result
- 처음에는 Gaussian noise처럼 보이지만 긴 시간 관찰하면 큰 jump가 자주 발견됨.
- 학습 시간이 이론만큼 오래걸리지 않는다. 
- Wide minima로 학습된 결과물들이 적지 않다. 

Proposed Framework
- Brownian motion 가정은 classical CLT에 기인한 것. (각 데이터에 의한 descent force가 평균내지는 것으로 이해)
- 하지만 finite-variance 가정은 생각보다 자주 깨짐. 
- 이 경우도 그러한 경우. Descent force들의 variance가 finite 하지 않은 것. 
- 그런 경우에는 Heavy-tailed random walk이 됨. 그것이 소위 말하는 Levy walk. 

Levy walk을 가정했을 때의 결과
- 더이상 Continous motion이 아니라 Discontinuity가 존재
- Minima를 탈출할 때 걸리는 시간이 더이상 Height에 의존하지 않음. 
- 대신 이제는 Width에 의존하게 됨. 

결과 정리
- Gradient noise는 Gaussian이 아니다
- Batch 크기는 생각보다 큰 영향이 없다. 
- 처음에는 Gradient noise가 크게 뛰어다니다가, 큰 움직임이 급격히 줄어들며 정확도가 갑자기 증가한다. (즉, 매우 초반에 정확도 상승이 생긴다)