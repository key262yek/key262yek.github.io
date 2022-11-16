---
title: "Brownian motion: introduction to the concept of path integration"
date: 2022-11-03
categories:
  - Review
  - Physics
tags:
  - Path Integral
  - Brownian motion
  - Wiener process
published: false
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

["Path Integrals in Physics"]({% post_url 2022-10-31-Path-Integral %})를 리뷰하는 thread 1-1장, Brown 운동을 통한 경로적분 개괄이다. 
해당 절은 Wiener measure와 Wiener integral의 존재성과 성질들을 유도하는 내용을 담고 있다. 

## Brownian motion of a free particle, diffusion equation and Markov chain

### Diffusion equation: Macroscopic approach
먼저 Brown 운동을 하는 입자들의 분포함수가 만족하는 방정식인 확산 방정식을 유도해보자. 
입자들의 밀도 함수를  $\rho(x, t)$라 했을 때 이들의 current 함수 $j(x)$는 밀도 함수의 공간 기울기에 비례한다.
\begin{equation}
    j(x) = - D \pdv{\rho(x, t)}{x}
\end{equation}
그리고 입자가 갑자기 생기거나 사라지지 않으므로, 특정 지점의 밀도의 시간변화율은 그 지점의 current의 공간 기울기로 주어진다. 
\begin{equation}
    \pdv{\rho(x,t)}{t} = -\pdv{j(x, t)}{x}
\end{equation}
이 두 식을 연립하면 아래와 같은 밀도 함수의 확산 방정식이 나온다.
\begin{equation}
    \pdv{\rho(x, t)}{t} = D \pdv[2]{\rho(x, t)}{x}
\end{equation}