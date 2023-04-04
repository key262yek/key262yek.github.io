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
count = (y <= np.exp(- np.exp(x))).sum()
area = L * (count / N)  
```
$[0,L] \times [0, 1]$ 구간에 uniform random number $(x, y)$를 $N$개 sampling하고 그 중 그래프 $y = e^{-x}$ 아래에 있는 점들의 수를 셈함으로써 전체 공간 대비 그래프 아래 넓이의 비율을 구할 수 있게 된 것이다. 

조금 다른 방법으로는 $y$의 sampling을 제거한 버젼이다. 
$x$가 결정된 상황에서 count하게 될 $y$ sample의 비율은 $e^{-e^x}$이므로, 이를 굳이 sampling하지 않고 기댓값으로 계산하는 방법론이다. 
```python
N = 100000
x = np.random.uniform(0, L, N)
p = np.exp(- np.exp(x)).sum() / N
area = L * p
```

하지만 이 방법을 이용하면 $N=10^4$ 정도에서 10% 정도의 오차가 생기고, 0.1% 수준의 오차로 답을 맞추기 위해서는 $N=10^6$ 정도는 되어야 한다. 
(앞선 방법보다는 두번째 방법이 수렴성이 조금 더 좋다.)
매우 느리게 수렴하는 것인데, 이는 Naive한 Monte Carlo method의 sampling이 비효율적이기 때문이다. 
부연설명하자면 우리가 적분하고자 하는 함수는 매우 빠르게 감소하는 함수이기 때문에 $x$가 일정값 이상인 영역에서의 sampling은 대부분 그래프 위에 존재할 것이고, 넓이 계산에 포함되지 않을 것이다. 
대부분의 넓이는 $x$가 작은 영역에서 계산되므로, 더 정확한 계산을 위해서는 $x$가 작은 영역에서의 sampling이 더 많이 일어나야 한다.

## First improvement: Importance sampling
이러한 Naive Monte Carlo method의 한계를 타파하는 방법으로는 Importance sampling이란 것이 있다. 
$x$가 작은 영역에 더 가중치를 두어 sampling하고 이를 토대로 넓이를 계산하는 방식이다. 
이를 위해 적분과 $x$의 sampling 사이 관계를 좀 더 고찰해보자면 이렇다.
\begin{equation}
  \int_0^L e^{-e^x} \dd x = L \times \int_0^L e^{-e^x} \frac{1}{L} \dd x = L \quad \mathbb{E}[e^{-e^x}]
\end{equation}
즉, Monte Carlo integration은 다름이 아니라 함수의 평균값을 구하는 문제와 동일하다. 

그렇기에 확률변수의 분포의 변화가 기댓값에 어떻게 영향을 주는지만 확인한다면 분포를 바꾸어 예측을 해볼 수 있다. 
확률분포 $p(x)$를 따르는 확률변수의 $f(x)$에 대한 기댓값은 확률분포 $q(x)$를 따르는 확률 변수의 $f(x) p(x) / q(x)$의 기댓값과 같다. 
\begin{equation}
  \mathbb{E}_{x \sim p} \qty[f(x)] 
  = \int f(x) p(x) \dd x 
  = \int f(x) \frac{p(x)}{q(x)} q(x) \dd x
  = \mathbb{E}_{x \sim q} \qty[f(x) \frac{p(x)}{q(x)}]
\end{equation}

지금의 경우에 대응해보자면 $p(x) = 1/L, f(x) = e^{-e^x}$인 것이고,
만약 $q(x) = e^{-x} / (1 - e^{-L})$라 하면 우리는 아래의 코드로 같은 적분을 수행할 수 있다. 
```python
N = 100000
C = 1 / (1 - np.exp(-L))
q = np.random.uniform(0, 1, N)
x = - np.log(1 - q / C)

s = (np.exp(- np.exp(x)) / (L * C * np.exp(-x))).sum() / N
area = L * s
```
이 경우 $N=10^6$에서 0.01% 오차로 더 정밀한 계산이 가능하다. 

## Metropolis algorithm
하지만 Importance sampling 역시 $q(x)$에 대해서 정확히 알고 있고, 이를 따라 sampling할 수 있어야한다는 한계가 있다.
복잡한 시스템에서는 확률분포를 정확히 계산하는 것도 어렵고, 이를 따르는 확률변수를 sampling하기는 더 힘들다.
그렇기 때문에 도입된 방법론이 Metropolis algorithm이고, 이를 발전시킨 것이 Metropolis-Hastings algorithm이다.
(Metropolis algorithm이라 불리는 초기 방법론도 Metropolis와 Hastings의 공동 연구로 나온 것이지만, 이후 Hastings가 발전시킨 방법론에 이르러 Hastings이름도 병기되기 시작했다.)

아이디어는 어떤 초기 분포 $p_0(x)$에서 출발하더라도 Markov chain sampling을 거치며 sampling distribution이 변화하도록 하고, 종국에는 원하는 분포 $p(x)$로 수렴하도록 하는 것이다. 
우리에게 필요한 정보는 원하는 분포 $p(x)$가 비례하는 것으로 알려진 비교적 간단한 함수 $w(x)$이다. (즉, $w(x)$를 모든 공간에 대해 적분해 Normalization factor를 구하는 과정이 어려운 경우에 쓰기 좋다)

이번에는 $w(x) = e^{-x}$라 하고, Normallization factor $C$를 모르는 상황에서 Metropolis algorithm을 이용해 적분을 계산해보자.
계산하고자 하는 함수는 그대로다. 
\begin{equation}
  \int_0^L e^{-e^x} \dd x 
  = \int_0^L \frac{e^{-e^x}}{C e^{-x}} C e^{-x} \dd x
  = \frac{1}{N} \sum_{i=1}^N \frac{e^{-e^{x_i}}}{C e^{-x_i}}, \quad X \sim C e^{-x}
\end{equation}
이 과정에서 우리가 모르는 부분은 두 가지이다.
$w(x)$에 비례하는 분포로 $X$를 sampling하는 방법과 $C$를 구하는 방법.
sampling 방법은 Metropolis algorithm을 소개하며 설명할 것이니 미뤄두고, $C$는 이 역시 적분을 근사하는 것으로 구할 수 있다.
\begin{equation}
  1 = \int_0^L \frac{1}{L} \dd x
  = \int_0^L \frac{1}{L C e^{-x}} C e^{-x} \dd x
  = \mathbb{E}\qty[\frac{1}{L C e^{-x}}] \\
\end{equation}

따라서 적분은 아래와 같은 식으로 정리된다.
\begin{equation}
  \int_0^L e^{-e^x} \dd x 
  = \frac{1}{N} \sum_{i=1}^N \frac{e^{-e^{x_i}}}{C e^{-x_i}}
  = L \frac{\sum_{i=1}^N e^{-e^{x_i}} / e^{-x_i}}{ \sum_{i = 1}^N 1 / e^{-x_i}}
\end{equation}

이를 계산하는 알고리즘은 다음과 같다. 
```python
def w(x):
    return np.exp(-x)

current = np.random.uniform(0, L)
samples = []
for _ in range(N + 100):
    samples.append(current)
    
    new = np.random.uniform(0, L)  # 1
    rate = w(new) / w(current)     # 2
    p = np.random.uniform(0, 1) 
    if p < rate:                   # 3
        current = new

samples = np.array(samples[100:])  # 4
C = (1 / np.exp(-samples)).sum()
S = (np.exp(-np.exp(samples)) / np.exp(-samples)).sum()
return L * S / C
```

Metropolis algorithm의 핵심은 주석을 단 1번부터 3번 줄에 있다. 
새로운 sample을 random하게 고르고(current에 의존하는 방식으로 sampling할 수도 있지만, 여기선 저 방법이 훨씬 오차가 적다) 새 sample을 선택할지 말지를 `rate`라는 변수를 통해 결정한다.
rate가 1보다 크면 항상 선택하고, 1보다 작은 경우 rate의 확률로 sample을 선택한다.
(즉, rate가 작을 수록 더 작은 확률로 sample이 renewal된다)
그리고 renewal 여부와 상관없이 `current`를 sample 집합에 포함시킨다.

이러한 과정을 따르게 되면 종국에는 sample들이 우리가 원하는 $w(x)$에 비례하는 분포를 띄는 확률변수가 된다. (초반에는 오차가 있을 수 있어 초반 100개의 결과는 제외했다)

## Metropolis-Hastings algorithm
Metropolis algorithm의 확장판인 MH algorithm은 새로운 sample candidate를 고르는 방법을 일반화한 것이라 할 수 있다.
앞서 소개한 알고리즘에서 새로운 sample candidate를 선택하는 방법은 아래와 같은데,
```python
new = np.random.uniform(0, L)   # case 1
new = current + np.random.uniform(-dx, dx) # case 2
```
두 방법 모두 전이 확률이 대칭적인 분포이다. 
즉, current가 $x$인 상황에서 $x'$이 candidate로 나오는 확률과 current가 $x'$인 상황에서 $x$가 candidate으로 나오는 확률이 같다.
하지만 항상 이런 경우만 있으리란 보장은 없으므로 비대칭적인 확률을 가지는 경우까지 확장한 경우가 Hastings가 제안한 방법이다. 

그 경우 sample candidate이 accept될 확률이 아래와 같이 바뀐다.
$$
\begin{align}
    A(x'|x) &= min\qty(1, \frac{w(x')}{w(x)}) \tag{Metropolis} \\
    A(x'|x) &= min\qty(1, \frac{w(x')p(x'\rightarrow x)}{w(x)p(x \rightarrow x')}) \tag{Hastings}
\end{align}
$$

이렇게 Acceptance probability를 조정하면, 비대칭적인 전이확률로 주어진 Markov chain sampling이라 하더라도 sampling의 분포는 $w(x)$에 비례하는 확률분포를 따르게 된다. 

## Detailed balance
이를 이해하기 위해서 우리는 sampling을 반복함에 따라 sample들이 따르는 확률분포가 어떻게 변하는지 추적할 필요가 있다. 
$n$번째 sample이 따르는 확률 분포를 $p_n(x)$라 하자. 
이를 통해 $n + 1$번째 sample의 확률분포 $p_{n+1}(x)$가 어떻게 주어질지 알아보자. 

$p_n(x)\dd x$의 확률로 주어진 $n$번째 sample $x$가 있을 때,
그 다음 sample $x'$은 $p(x \rightarrow x')$의 확률로 결정된다. 
그리고 그 sample이 accept 될 확률은 $A(x'|x)$이다.
따라서 $p_{n}(x)$에서 $\int \dd x' p(x \rightarrow x') A(x' | x) p_n(x)$ 만큼의 확률은 다음 sample에서 다른 값의 확률로 옮겨간다.
반대로 생각하면 다른 위치 $x'$에서도 $x$ 위치로의 확률 이동이 존재한다.
그 값은 $\int \dd x' p(x' \rightarrow x) A(x | x) p_n(x')$이 된다.
따라서 $p_n(x)$의 점화식은 아래와 같이 주어진다.

\begin{equation}
  p_{n+1}(x) = p_n(x) + \int \dd x' (p(x' \rightarrow x) A(x|x') p_n(x') - p(x \rightarrow x') A(x'|x) p_n(x))
\end{equation}

이 점화식을 완전히 푸는 것은 쉽지 않다.
하지만 이 해의 극한이 어떤 평형 조건을 만족할지는 구할 수 있다. 
\begin{equation}
  p(x' \rightarrow x) A(x|x') p(x') = p(x \rightarrow x') A(x'|x) p(x)
\end{equation}

따라서 평형 상태에서의 두 지점의 확률분포 비율 $p(x')/p(x)$는 아래와 같이 주어진다. 
\begin{equation}
  \frac{p(x')}{p(x)}
  = \frac{p(x \rightarrow x') A(x'|x)}{p(x' \rightarrow x) A(x|x')}
\end{equation}

이제 우리가 알아야 하는 것은 Accetance probability의 비율이다. 
만약 $w(x')p(x' \rightarrow x) > w(x) p(x \rightarrow x')$라면
$A(x'|x) = 1$로 주어지므로 
\begin{equation}
  \frac{A(x' | x)}{A (x | x')} = \frac{1}{A(x|x')} = \frac{w(x')p(x' \rightarrow x)}{w(x)p(x \rightarrow x')}
\end{equation}
이 되고, 반대의 경우도 최종 결과 값은 동일하다. 
따라서 앞선 식에 이를 대입하면 우리는 평형 분포의 비율을 구할 수 있다.
\begin{equation}
  \frac{p(x')}{p(x)}
  = \frac{p(x \rightarrow x')}{p(x' \rightarrow x)}\frac{w(x')p(x' \rightarrow x)}{w(x)p(x \rightarrow x')}
  = \frac{w(x')}{w(x)}
\end{equation}

## Conclusion
이렇게 MH algorithm의 복잡한 sampling 방법이 어떻게 원하는 분포를 따라 sampling을 할 수 있게 되었는지 확인하였다. 
정리하자면 아래와 같은 부분을 확인할 수 있었다.

1. MH algorithm의 사용처: 확률분포의 함수 꼴은 알지만 normalization factor를 계산하는 것은 매우 어려울 때
2. MH algorithm의 알고리즘 핵심: New sample을 reject하는 acceptance probability
3. MH algorithm의 작동원리: 분포의 점화식의 극한이 이루게 되는 detailed balance가 우리가 원하는 확률분포를 이루도록 강제하는 것.







