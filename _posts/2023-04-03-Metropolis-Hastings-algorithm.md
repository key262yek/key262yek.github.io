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


확률밀도함수 $p(x)$를 완전히 아는 상황에서 1차원 확률변수를 sampling하는 가장 간단한 방법은 무엇일까요?
누적분포함수 $C(x) = P(X \leq x)$를 이용하는 방법입니다.
$[0,1]$ 구간의 uniform random number $p$를 뽑은 후에 $C(x) = p$인 $x$를 sample로 하면 이 sample $x$가 확률분포 $p(x)$를 따른다는 점을 이용하는 것이죠. 

하지만 만약 확률밀도함수 $p(x)$를 잘 알지 못하거나, 확률변수의 차원이 너무 커서 $C(x)$와 $p$의 대응을 알고리즘적으로 쉽게 유추하지 못할 때는 어떻게 해야할까요?
그럴때 사용되는 방법이 `Markov Chain Monte Carlo(MCMC)` 방법론입니다. 
MCMC의 자세한 정의는 다른 글을 통해 보충하도록 하고, 오늘은 그 중에서 `Metropolis-Hastings(MH) algorithm`의 작동방식에 대해서 알아보겠습니다. 

## Purpose of algorithm
여느 Monte Carlo method와 같이 MH algorithm 역시 기본적으로 sampling 기법입니다. 
모분포 $p(x)$를 정확히 구할 수는 없지만 $p(x)$가 어떤 함수 $f(x)$에 비례한다는 것을 안다거나, 
혹은 두 정의역 $x_1, x_2$에 대해서 존재 확률비 $p(x_1)/p(x_2)$를 구하기 쉬운 환경에서 MH algorithm을 이용하면 $p(x)$를 따르는 확률변수를 sampling할 수 있습니다. 

MH algorithm은 단일 단계로 작동하는 알고리즘이 아닙니다.
처음에 어떤 prior probability를 기반으로 시작했던 간에, 반복적으로 sampling하면서 sample이 따르는 확률분포가 $p(x)$로 근접해가도록 강제하는 알고리즘이죠. 
이 과정에서 $n$번째 sample에 대한 정보에만 의존해 $n + 1$번째 sample을 만들어내기 때문에 Markov chain이란 이름이 앞에 붙어 Markov Chain Monte Carlo method로 불리는 것입니다. 

## Toy model: $\int_0^L e^{- e^x} \dd x$
바로 MH algorithm으로 넘어가기 전에 기초적인 Monte Carlo method부터 살펴봅시다. 
가장 쉬운 예시는 적분을 random sampling을 이용해 구하는 예시입니다. 
$\int_0^L e^{- e^x} \dd x$라는 적분을 하려고 할 때, 가장 기본적인 Monte Caro method는 아래와 같이 구현됩니다. 
```python
N = 100000
x = np.random.uniform(0, L, N)
y = np.random.uniform(0, 1, N)
count = (y <= np.exp(- np.exp(x))).sum()
area = L * (count / N)  
```
$[0,L] \times [0, 1]$ 구간에 uniform random number $(x, y)$를 $N$개 sampling하고 그 중 그래프 $y = e^{-e^x}$ 아래에 있는 점들의 수를 셈함으로써 전체 공간 대비 그래프 아래 넓이의 비율을 구할 수 있고, 이를 전체 넓이 $L$에 곱해 넓이를 구하는 방식입니다.

조금 개선된 방법은 $y$의 sampling을 평균치로 치환한 방법입니다. 
$x$가 결정된 상황에서 그래프 아래 넓이에 포함되는 $y$ sample의 비율은 $e^{-e^x}$이 되므로, 이를 굳이 sampling해 오차를 키우지 않고 기댓값을 바로 계산해 더할 수 있습니다. 
```python
N = 100000
x = np.random.uniform(0, L, N)
p = np.exp(- np.exp(x)).sum() / N
area = L * p
```

하지만 이 방법들을 이용했을 때 $N=10^4$ 정도에서는 10% 정도의 오차로 적분을 근사할 수 있고, 0.1% 수준의 오차로 답을 맞추기 위해서는 $N=10^6$ 정도 되어야 합니다. 
(앞선 방법보다는 두번째 방법이 수렴성이 조금 더 좋습니다.)
매우 느리게 수렴하는 것인데, 이는 Naive한 Monte Carlo method의 sampling이 비효율적이기 때문입니다. 
부연설명하자면 우리가 적분하고자 하는 함수는 매우 빠르게 감소하는 함수이기 때문에 $x$가 큰 영역에서의 sampling은 대부분 그래프 위에 존재해 넓이 계산에 포함되지 않는 무의마한 sample이기 때문입니다. 
그래프 넓이의 대부분은 $x$가 작은 영역이므로, 더 정확한 계산을 위해서는 $x$가 작은 영역에서의 sampling이 더 많이 일어나야 합니다.

## First improvement: Importance sampling
이러한 Naive Monte Carlo method의 한계를 타파하는 방법이 Importance sampling입니다. 
$x$가 작은 영역이 더 빈번히 나오도록 sampling하고 이를 토대로 넓이를 계산합니다. 
이를 위해 적분과 $x$의 sampling 분포 사이 관계를 좀 더 고찰해봅시다.
\begin{equation}
  \int_0^L e^{-e^x} \dd x = L \times \int_0^L e^{-e^x} \frac{1}{L} \dd x = L \; \mathbb{E}[e^{-e^x}]
\end{equation}
Monte Carlo integration은 다름이 아니라 함수의 평균값을 구하는 문제라 할 수 있습니다. 

그렇기에 확률 분포의 변화가 기댓값에 어떻게 영향을 주는지만 확인한다면 분포를 바꾸어도 같은 기댓값을 가지도록 함수를 변화시킬 수 있습니다.
정확히는, 확률분포 $p(x)$를 따르는 확률변수의 $f(x)$에 대한 기댓값은 확률분포 $q(x)$를 따르는 확률 변수의 $f(x) p(x) / q(x)$의 기댓값과 같습니다. 

$$
\begin{equation}
  \mathbb{E}_{x \sim p} \qty[f(x)] 
  = \int f(x) p(x) \dd x 
  = \int f(x) \frac{p(x)}{q(x)} q(x) \dd x
  = \mathbb{E}_{x \sim q} \qty[f(x) \frac{p(x)}{q(x)}]
\end{equation}
$$

지금의 경우에 대응해보자면 $p(x) = 1/L, f(x) = e^{-e^x}$이고,
만약 $q(x) = e^{-x} / (1 - e^{-L})$로 설정한다면 우리는 아래의 코드로 같은 적분을 수행할 수 있습니다. 
```python
N = 100000
C = 1 / (1 - np.exp(-L))
q = np.random.uniform(0, 1, N)
x = - np.log(1 - q / C)

s = (np.exp(- np.exp(x)) / (L * C * np.exp(-x))).sum() / N
area = L * s
```
이 경우 $N=10^6$에서 0.01% 오차로 더 정밀한 근사가 가능합니다. 

## Metropolis algorithm
하지만 Importance sampling 역시 $p(x), q(x)$에 대해서 정확히 알고 있고, 이를 따라 sampling할 수 있어야한다는 제한이 있습니다.
복잡한 시스템에서는 확률분포를 정확히 계산하는 것도 어렵고, 이를 따르는 확률변수를 sampling하기는 더 힘듭니다.
그렇기 때문에 도입된 방법론이 Metropolis algorithm이고, 이를 발전시킨 것이 Metropolis-Hastings algorithm입니다.
(Metropolis algorithm이라 불리는 초기 방법론도 Metropolis와 Hastings의 공동 연구로 나온 것이지만, 이후 Hastings가 발전시킨 방법론에 이르러 Hastings이름도 병기되기 시작했습니다)

아이디어는 어떤 초기 분포 $p_0(x)$에서 출발하더라도 Markov chain sampling을 거치며 sampling 분포가 점차 원하는 분포 $p(x)$로 수렴하도록 하는 것입니다. 
이때 우리에게 필요한 정보는 원하는 분포 $p(x)$가 비례하는 함수 $w(x)$입니다. (즉, $w(x)$를 모든 공간에 대해 적분해 Normalization factor를 구하는 과정이 어려운 경우에 쓰기 좋습니다)

이번에는 $w(x) = e^{-x}$라 하고, Normallization factor $C$를 모르는 상황에서 Metropolis algorithm을 이용해 적분을 계산해봅시다.
계산하고자 하는 함수는 그대로입니다. 
\begin{equation}
  \int_0^L e^{-e^x} \dd x 
  = \int_0^L \frac{e^{-e^x}}{C e^{-x}} C e^{-x} \dd x
  = \frac{1}{N} \sum_{i=1}^N \frac{e^{-e^{x_i}}}{C e^{-x_i}}, \quad X \sim C e^{-x}
\end{equation}
이 과정에서 우리가 모르는 부분은 두 가지입니다.
$w(x)$에 비례하는 분포로 $X$를 sampling하는 방법과 Normalization factor $C$를 구하는 방법.
sampling 방법은 Metropolis algorithm을 소개하며 설명할 것이니 미뤄두고, $C$는 역시 적분을 근사하는 것으로 구할 수 있습니다.

$$
\begin{align}
  1 &= \int_0^L \frac{1}{L} \dd x
  = \int_0^L \frac{1}{L C e^{-x}} C e^{-x} \dd x
  = \mathbb{E}\qty[\frac{1}{L C e^{-x}}] \\
  C &= \mathbb{E} \qty[\frac{1}{L e^{-x}}] = \frac{1}{NL} \sum_{i=1}^N \frac{1}{e^{-x_i}}
\end{align}
$$

따라서 적분은 아래와 같은 식으로 정리되고,
\begin{equation}
  \int_0^L e^{-e^x} \dd x 
  = \frac{1}{N} \sum_{i=1}^N \frac{e^{-e^{x_i}}}{C e^{-x_i}}
  = L \frac{\sum_{i=1}^N e^{-e^{x_i}} / e^{-x_i}}{ \sum_{i = 1}^N 1 / e^{-x_i}}
\end{equation}
이를 계산하는 알고리즘은 다음과 같습니다.
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

Metropolis algorithm의 핵심은 주석을 단 1번부터 3번 줄에 있습니다. 
새로운 sample을 random하게 고르고(current에 의존하는 방식으로 sampling할 수도 있지만, 여기선 저 방법이 훨씬 오차가 적습니다) 새 sample을 선택할지 말지를 `rate`라는 변수를 통해 결정합니다.
rate가 1보다 크면 항상 새로운 sample을 accept하고, 1보다 작은 경우 rate의 확률로 sample을 accept합니다.
(즉, rate가 작을 수록 더 작은 확률로 sample이 renewal되게 됩니다)
그리고 renewal 여부와 상관없이 `current`를 sample 집합에 포함시킵니다.

이러한 과정을 반복하게 되면 sample은 우리가 원하는 $w(x)$에 비례하는 분포를 띄는 확률변수가 됩니다. 다만 초반에는 초기화 분포의 영향에 따른 오차가 있을 수 있기 때문에 주석 4번줄처럼 초반의 결과를 제외해야 합니다.

## Metropolis-Hastings algorithm
Metropolis algorithm의 확장판인 MH algorithm은 새로운 sample candidate를 고르는 방법을 일반화한 것이라 할 수 있습니다.
앞서 소개한 알고리즘에서 새로운 sample candidate를 선택하는 방법은 아래와 같은데,
```python
new = np.random.uniform(0, L)   # case 1
new = current + np.random.uniform(-dx, dx) # case 2
```
두 방법 모두 전이 확률이 대칭적인 분포인 것을 알 수 있습니다. 
(current가 $x$인 상황에서 $x'$이 candidate로 나오는 확률과 current가 $x'$인 상황에서 $x$가 candidate으로 나오는 확률이 같을 때, 대칭적인 분포라 부릅니다)
하지만 항상 대칭적인 분포로 sampling하란 법은 없으므로 비대칭적인 분포까지 확장한 경우가 Hastings가 제안한 MH algorithm입니다. 

Hastings algorithm에서는 acceptance probability만 아래와 같이 바뀝니다.

$$
\begin{align}
    A(x'|x) &= min\qty(1, \frac{w(x')}{w(x)}) \tag{Metropolis} \\
    A(x'|x) &= min\qty(1, \frac{w(x')p(x'\rightarrow x)}{w(x)p(x \rightarrow x')}) \tag{Hastings}
\end{align}
$$

이렇게 Acceptance probability를 조정하면, 비대칭적인 전이확률로 주어진 Markov chain sampling이라 하더라도 sampling의 분포는 $w(x)$에 비례하는 확률분포를 따르게 됩니다. 

## Detailed balance
왜 MH algorithm을 반복하다 보면 sample 분포가 원하는 분포로 수렴하게 되는 것일까요?
이를 이해하기 위해서 우리는 sample 분포가 sampling을 반복함에 따라 어떻게 변하는지 추적할 필요가 있습니다. 
$n$번째 sample 분포를 $p_n(x)$라 할 때, $n + 1$번째 sample 분포 $p_{n+1}(x)$가 어떻게 주어질지 알아봅시다. 

$p_n(x)\dd x$의 확률로 주어진 $n$번째 sample $x$가 있을 때,
그 다음 sample의 값이 $x'$로 주어질 확률을 $p(x \rightarrow x') \dd x'$라 합시다. 
그리고 그 sample이 진짜 다음 sample로 accept 될 확률은 $A(x'|x)$입니다.
따라서 $p_{n}(x) \dd x$에서 $p(x \rightarrow x') A(x' | x) p_n(x) \dd x \dd x'$ 만큼의 확률은 다음 sample로 $x'$이 나올 확률이 됩니다.

반대로 생각하면 다른 위치 $x'$에서도 $x$로의 확률 이동이 존재합니다.
그 값은 $p(x' \rightarrow x) A(x | x) p_n(x') \dd x \dd x'$이죠.
확률 flow를 모든 $x'$에 대해 적분하면 $p_n(x)$의 알짜 변화량을 구할 수 있게 되고, 
이로부터 $p_n(x)$의 점화식을 구할 수 있습니다.

\begin{equation}
  \hspace{-30pt}
  p_{n+1}(x) = p_n(x) + \int \dd x' (p(x' \rightarrow x) A(x|x') p_n(x') - p(x \rightarrow x') A(x'|x) p_n(x))
\end{equation}

이 점화식을 완전히 푸는 것은 쉽지 않습니다.
하지만 해의 극한이 어떤 평형 조건을 만족할지는 쉽게 보입니다. 
\begin{equation}
  p(x' \rightarrow x) A(x|x') p(x') = p(x \rightarrow x') A(x'|x) p(x)
\end{equation}
따라서 평형 상태에서의 두 지점의 확률분포비 $p(x')/p(x)$는 아래와 같이 결정됩니다. 
\begin{equation}
  \frac{p(x')}{p(x)}
  = \frac{p(x \rightarrow x') A(x'|x)}{p(x' \rightarrow x) A(x|x')}
\end{equation}

이제 우리가 알아야 하는 것은 Accetance probability의 비율입니다. 
만약 $w(x')p(x' \rightarrow x) > w(x) p(x \rightarrow x')$라면
$A(x'|x) = 1$로 주어지므로 
\begin{equation}
  \frac{A(x' | x)}{A (x | x')} = \frac{1}{A(x|x')} = \frac{w(x')p(x' \rightarrow x)}{w(x)p(x \rightarrow x')}
\end{equation}
이 되고, 반대의 경우도 최종 결과 값은 동일합니다. 
앞선 식에 이를 대입하면 우리는 평형 분포가 $w(x)$의 비율로 주어짐을 알 수 있고, 평형 분포를 따르는 sample은 자연스레 우리가 원하는 분포를 따르게 되는 것입니다.
\begin{equation}
  \frac{p(x')}{p(x)}
  = \frac{p(x \rightarrow x')}{p(x' \rightarrow x)}\frac{w(x')p(x' \rightarrow x)}{w(x)p(x \rightarrow x')}
  = \frac{w(x')}{w(x)}
\end{equation}

## Conclusion
이렇게 MH algorithm의 복잡한 sampling 방법이 어떻게 원하는 분포를 따르는지 알아봤습니다. 

1. MH algorithm의 사용처: 확률분포의 함수 꼴은 알지만 normalization factor를 계산하는 것은 매우 어려울 때
2. MH algorithm의 알고리즘 핵심: New sample을 reject하는 acceptance probability의 정의
3. MH algorithm의 작동원리: 평형 분포가 이루게 되는 detailed balance가 우리가 원하는 확률분포를 이루도록 강제하는 것.







