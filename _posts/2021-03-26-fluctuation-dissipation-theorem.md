---
title: "Fluctuation Dissipation Theorem"
date: 2021-03-26T21:35:30-1:00
categories:
  - Statistical Physics
tags:
  - Fluctuation Dissipation Theorem
  - R.Kubo
published: false
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

Fluctuation Dissipation Theorem(FDT)은 비평형 통계물리에서 매우 다양한 형태로 나타나는 정리입니다.
가장 유명한 형태는 다른 외력 없이, 유체에 의한 마찰력과 random force만 받아 움직이는 입자의 확산계수 $D$가 마찰계수 $\gamma$와 만족해야하는 Einstein Relation이 있습니다.
\begin{equation}
    D \gamma = k_B T
\end{equation}
입자 운동의 Fluctuation의 크기에 해당하는 $D$와 Dissipation과 관계 있는 마찰계수 $\gamma$의 관계라 하여 Flcutation Dissipation Theorem이라 하는데요.
이에 대해 잘 정리된 R.Kubo의 [강의 자료](https://iopscience.iop.org/article/10.1088/0034-4885/29/1/306)가 있어서 이를 읽으며 정리해보고자 합니다.

### Introduction

유체 속에 떠다니는 콜로이드 입자의 운동은 가장 대표적인 브라운 운동의 예시입니다.
이런 입자는 유체 분자들의 충돌에 의해 크게 두 가지 영향을 받게 되는데요.
하나는 마찰력이고, 다른 하나는 random force입니다.
한 가지 원인에서 유도된 현상이다 보니 마찰력과 random force는 특정한 관계를 가져야할 것이고, 
이 관계를 우리는 fluctuation-dissipation theorem이라 합니다.

FDT는 일반적으로 외부 자극에 대한 반응과 자극이 없는 상황에서의 내재적 요동 사이 관계로 쓰여집니다.
여기서 반응이란 보통 반응 함수나, 수용률, 임피던스와 같은 값으로 특정되고,
요동은 열평형 상태에서 특정한 물리량의 correlation function, 혹은 이들의 spectrum으로 주어지게 됩니다. 그렇기 때문에 FDT는 임피던스로부터 요동을 예측하는 형태거나(Nyquist theorem), 요동과 수용률 사이 관계를 밝히는 형태(Onsager's symmetry)로 나타납니다.

내용만 봤을 때 매우 중요하고 요긴한 정리인데도 불구하고, FDT는 주목을 늦게 받았습니다. 그도 그럴것이, 비평형 통계물리에서는 볼츠만 방정식이 거의 유일한 도구인줄 안 시간이 길었거든요. Einstein, Nyquist, Onsager가 1900년대 초반에 이미 FDT와 같은 맥락의 연구를 진행한 역사는 있었지만, 제대로된 연구는 1950년대 들어서 Callen and Welton (1951), Callen and Greene (1952), Takahashi (1952), Kubo and Tomita (1954) 등의 연구가 있었던 1950년대 되어서야 이뤄졌습니다.

### Einstein relation

Einstein relation은 FDT의 예시로 가장 많이 들어지는 관계식인데요.
Potential field $V$에 의해 만들어지는 유속 $u_d$가 
\begin{equation}
    u_d = - \frac{1}{\gamma} \dv{V}{x}
\end{equation}
로 주어지는데, 여기에 밀도 분포 $\rho(x)$의 변화에 따라 만들어지는 diffusion current까지 포함시켜 구한 유체의 유동은
\begin{equation}
    j(x) = - D\pdv{f(x)}{x} + u_d f(x)
\end{equation}
가 됩니다. 
평형상태에서 밀도 함수가 potential field에 의해 볼츠만 인자에 비례해야 한단 점 $f(x) \propto \exp(-V / k_B T)$, 그리고 알짜 유동이 0이 되어야 한단 점을 합치면
\begin{equation}
    D \gamma = k_B T
\end{equation}
를 만족해야한단 것을 유도할 수 있습니다.

이것을 현대적인 FDT의 형태로 옮겨오기 위해서 확산계수의 정의를 다시 살펴봅시다.
\begin{equation}
    D = \lim_{t \rightarrow \infty} \frac{1}{2t} \expval{\qty{x(t) - x(0)}^2}
\end{equation}
여기서 평균 $\expval{\cdot}$은 열평형 하에서의 앙상블 평균을 의미합니다.
즉, 긴 시간동안 평균적으로 퍼져나간 정도를 나타내는 물리량이라 할 수 있는데요.
변위를 속도를 이용해 표현하면
\begin{equation}
    x(t) - x(0) = \int_{0}^t u(t') \dd t'
\end{equation}
이 되고, 이를 확산계수 정의에 대입해주면 아래와 같은 속도의 autocorrelation function으로 정리된다.
\begin{align}
    D 
    &= \lim_{t \rightarrow \infty} \frac{1}{2t} \int_0^t \dd t_1 \int_0^t \dd t_2 \expval{u(t_1)u(t_2)}  \notag \newline
    &= \lim_{t \rightarrow \infty} \frac{1}{t} \int_0^t \dd t_1 \int_0^{t - t_1} \dd t_2 \expval{u(t_1)u(t')}  \newline
    &= \int_0^\infty \expval{u(t_0) u(t_0 + t)}\dd t \notag
\end{align}
이런 관점에서 Einstein relation은 마찰계수 $\gamma$가 브라운 운동의 flcutuation과 관련이 있다는, fluctuation-dissipation theorem의 일환이란 것을 확인할 수 있습니다.

### 

## References
