---
title: "Metropolis Hastings algorithm"
date: 2023-04-03
categories:
  - Programming
tags:
  - Markov Chain Monte Carlo
  - MCMC
published: true
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

## Introduction

확률밀도함수 $p(x)$를 완전히 아는 상황에서 1차원 확률변수를 sampling하는 가장 간단한 방법은 무엇일까?
누적분포함수 $C(x) = P(X \leq x)$를 이용하는 것이다.
$[0,1]$ 구간의 uniform random number $p$를 뽑은 후에 $C(x) = p$인 $x$를 sample로 하면 이 sample $x$는 필연적으로 확률분포 $p(x)$를 따르게 된다. 

하지만 만약 확률밀도함수 $p(x)$를 잘 알지 못하거나, 확률변수의 차원이 너무 커서 $C(x)$와 $p$의 대응을 알고리즘적으로 쉽게 유추하지 못할 때는 어떻게 해야할까?
그럴때 사용되는 방법이 `Markov Chain Monte Carlo(MCMC)` 방법론이다. 
MCMC의 자세한 정의는 다른 글을 통해 보충하도록 하고, 오늘은 그 중에서 `Metropolis-Hastings(MH) algorithm`의 작동방식에 대해서 설명하고자 한다. 

## Purpose of algorithm
여느 Monte Carlo method와 같이 MH algorithm 역시 기본적으로 sampling 기법이다. 
모분포 $p(x)$를 정확히 구할 수는 없지만 $p(x)$가 어떤 함수 $f(x)$에 비례한다는 것을 안다거나, 
혹은 두 확률변수 $x_1, x_2$에 대해서 두 확률 $p(x_1)/p(x_2)$를 구하기 쉬운 환경에서 MH algorithm을 이용하면 $p(x)$를 따르는 확률변수를 sampling할 수 있다. 

MH algorithm은 단일 단계로 작동하는 알고리즘이 아니다.
처음에 어떤 prior probability를 기반으로 시작했던 간에 반복적으로 sampling하면서 종국에는 sample이 따르는 확률분포가 $p(x)$로 근접해가도록 강제하는 알고리즘이다. 
이 과정에서 $n$번째 sample에 대한 정보를 이용해 $n + 1$번째 sample을 만들어내기 때문에 Markov chain이란 이름이 앞에 붙어 Markov chain Monte Carlo method로 불리는 것이다. 

## Toy model: $\int_0^L e^{- e^x} \dd x$
바로 MH algorithm으로 넘어가기 전에 기초적인 Monte Carlo method부터 살펴보자. 
가장 쉬운 예시는 적분을 random sampling을 이용해 구하는 예시이다. 
$\int_0^L e^{- e^x} \dd x$라는 적분을 하려고 할 때, 가장 기본적인 Monte Caro method는 아래와 같은 형태를 띈다. 
```python
N = 100000
x = np.random.uniform(0, L, N)
y = np.random.uniform(0, 1, N)
count = (y <= np.exp(- np.exp(x)).sum()
area = L * (count / N)  
```
$[0,L] \times [0, 1]$ 구간에 uniform random number $(x, y)$를 $N$개 sampling하고 그 중 그래프 $y = e^{-x}$ 아래에 있는 점들의 수를 셈함으로써 전체 공간 대비 그래프 아래 넓이의 비율을 구할 수 있게 된 것이다. 
하지만 이 방법을 이용하면 $N=10^4$ 정도에서 10% 정도의 오차가 생기고, 0.1% 수준의 오차로 답을 맞추기 위해서는 $N=10^8$ 정도는 되어야 한다. 
매우 느리게 수렴하는 것인데, 이는 Naive한 Monte Carlo method의 sampling이 비효율적이기 때문이다. 
부연설명하자면 우리가 적분하고자 하는 함수는 매우 빠르게 감소하는 함수이기 때문에 $x$가 일정값 이상인 영역에서의 sampling은 대부분 그래프 위에 존재할 것이고, 넓이 계산에 포함되지 않을 것이다. 
대부분의 넓이는 $x$가 작은 영역에서 계산되므로, 더 정확한 계산을 위해서는 $x$가 작은 영역에서의 sampling이 더 많이 일어나야 한다.

## First improvement: Importance sampling
이러한 Naive Monte Carlo method의 한계를 타파하는 방법으로는 Importance sampling이란 것이 있다. 
$x$가 작은 영역에 더 가중치를 두어 sampling하고 이를 토대로 넓이를 계산하는 방식이다. 
$x$가 뽑히는 확률분포가 $f(x) = C e^{-x}$꼴이라 가정해보자. ($C = (1 - e^{-L})^{-1}$)
그렇다면 $x$가 작은 영역은 기존보다 더 높은 확률로 count 될 것이다. 
왜곡을 다시 없애려면 $y$를 sampling해 count할 확률이 $f(x)^{-1} = e^{x}/C$배 되어야 한다.
알고리즘으로 표현해보면 아래와 같다. 

```python

```







