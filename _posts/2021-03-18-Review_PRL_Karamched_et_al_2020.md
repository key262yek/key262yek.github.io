---
title: "집단내 다양성이 결정의 속도와 정확성에 주는 영향"
date: 2021-03-18T2:35:30-04:00
categories:
  - Statistical Physics
tags:
  - Random Target Searching
  - Heterogeneity
  - Bhargav Karamched
  - Zachary P. Kilpatrick
  - Krešimir Josić
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

제가 박사과정에서 주로 연구하고 있는 분야는 'Random Target Searching'이란 이름의, 무작위적으로 움직이는 객체가 목표를 찾는 문제입니다.
객체가 움직이는 공간, 무작위적으로 움직이는 규칙, 목표의 물리적 의미에 따라 매우 다양한 문제를 포괄할 수 있는 분야인데요. 
이번에 읽은 논문 "Heterogeneity Improves Speed and Accuracy in Social Networks"[^1]는 2020년 11월 *Physics Review Letters*에 게재된 글로, 집단의 의사결정에서 다양성이 결정의 신속도와 정확도에 어떤 영향을 미치는지 Random Target Searching 문제를 통해 확인한 논문입니다.

[^1]: [paper link](https://journals.aps.org/prl/pdf/10.1103/PhysRevLett.125.218302)

# 저자 소개
저자가 많아 1저자만 간단히 살펴보겠습니다. 
주 저자인 Bhargav Karamched는 University of Oklahoma에서 생화학과 수학을 전공하고, University of Utah에서 2012년 수학으로 박사학위를 받았습니다. 현재는 이 논문의 공저자인 University of Houston의 Krešimir Josić, Will Ott 연구팀에서 박사 후 연구원으로 일하고 있습니다.

주된 연구분야는 편미분방정식, 확률미분방정식, 비평형 통계물리 등의 기법을 이용해 생물학, 생화학, 생물물리, 의사결정 등의 다양한 분야의 문제들에 관심을 가지는 것 같습니다. 
[개인 페이지](https://www.math.uh.edu/~bhargav/)에서 소개하고 있는 최근 관심 분야는 아래와 같습니다. 

* Developing mathematical models of synthetic gene circuits
* Developing mathematical models of emergent properties in bacterial consortia
* Evidence accumulation and decision making in networks
* Shear-induced chaos in glucose-insulin dynamics
* Developing mathematical models of axonal length sensing

# 논문 요약

# 논문 내용 상세설명

## Introduction

집단 의사결정은 인간 뿐 아니라 다양한 분야에서 찾아볼 수 있습니다. 예를 들어 아르헨티나 개미가 다른 개미를 뒤따라가며 하나의 경로를 만드는 과정이라던가[^2], 아프리카 야생견 집단이 주변 개들의 짖음에 반응해 출발 신호를 인식한다던지[^3] 하는 현상을 관찰할 수 있죠.

[^2]: A. Perna, B. Granovskiy, S. Garnier, S. C. Nicolis, M. Labe ́dan, G. Theraulaz, V. Fourcassie ́, and D. J. T. Sumpter, [PLoS Comput. Biol. 8, e1002592 (2012)](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1002592)

[^3]: R. H. Walker, A. J. King, J. W. McNutt, and N. R. Jordan, [Proc. R. Soc. B 284, 20170347 (2017)](https://doi.org/10.1098/rspb.2017.0347)

'개별 의사결정자들은 각자 가지고 있는 정보와 집단이 제공하는 정보를 어떻게 결합해 의사결정 할 수 있을까?'하는 질문의 답을 찾기 위해 연구진들은 세부적인 내용을 모두 추적할 수 있을만큼 작은 네트워크에서의 집단적 의사결정을 분석한 바 있습니다.[^4]
이 논문에서는 앞선 논문을 좀 더 크고, 비균질적인 네트워크로 확장한 연구라 할 수 있는데요.
간단히 정리하자면, 균질적인 네트워크에서는 잘못된 초기 의사결정이 잘못된 집단 의사결정으로 연결될 확률이 50%에 가까웠지만, 비균질적 네트워크에서는 초반의 의사결정은 어차피 섣부른 의사결정자들에게만 영향을 주게 되어 신중한 의사결정자들은 올바른 의사결정을 할 수 있는 상황이 만들어진단 것을 보였습니다.
이는 집단을 다양하게 구성하는 것이 (때때로 그들이 신뢰할 수 없거나, 틀리더라도) 보다 나은 선택일 수 있음을 시사합니다.

[^4]:  B. Karamched, S. Stolarczyk, Z. P. Kilpatrick, and K. Josić, [SIAM J. Appl. Dyn. Syst. 19, 1884 (2020)](https://doi.org/10.1137/19M1283793).

## Theory

### Model description

기존의 집단적 의사결정 모델들은 증거 수집의 시간적 변화를 무시하거나[^5], 합리적 의사결정자에 대한 설명이 부재했습니다[^6]. 이 논문의 모델에서는 두 가지를 모두 포함시켰고, 이로 말미암아 비합리적 의사결정자에 대한 이해를 도모했습니다.

[^5]: R. P. Mann, [Proc. Natl. Acad. Sci. U.S.A. 115, E10387 (2018)](https://doi.org/10.1073/pnas.1811964115)

[^6]: D. J. Watts, [Proc. Natl. Acad. Sci. U.S.A. 99, 5766 (2002)](https://doi.org/10.1073/pnas.082090499)

모델의 특징은 다음과 같습니다. 

* All-to-all network
* 각 의사결정자는 두 가지 선택지 중 하나를 선택해야 한다
* 개인적으로 정보를 모으기도 하고(private observation), 다른 의사결정자의 결정에게 영향을 받기도 한다.

### 신뢰도 (Belief)

각각의 의사결정자는 정보를 모아서 선택지에 대한 신뢰도(belief) $y_i(t)$를 변화시킵니다.
$i$번째 의사결정자가 시간 t 동안 관찰한 정보들을 $\xi_{1:t}^i$라 했을 때, 신뢰도는 그 정보를 기반으로 두 선택지(논문에서는 $H^+$와 $H^-$로 표기) 각각이 옳을 확률의 비율의 로그값으로 정의했는데요.

\begin{equation}
    y_i(t) = log(P(H^+ | \xi_{1:t}^i) / P(H^- | \xi_{1:t}^i))
    \label{eq: definition of belief}
\end{equation}

이런 정의 위에서, 시간에 따라 주어지는 정보들이 서로 상관관계가 없고, 그 주기가 매우 짧아 거의 연속적으로 정보를 얻는다 가정하면 신뢰도는 아래의 Langevin equation을 따르는 변수로 근사할 수 있습니다.

\begin{equation}
    \dd y_i = \pm \alpha \dd t + \sqrt{2\alpha} \dd W_i,
    \label{eq : langevin equation of belief}
\end{equation}

여기서 첫번째 항은 옳은 선택지를 향하는 drift 항으로, 옳은 선택지가 $H^+$냐 $H^-$냐에 따라 부호가 달라집니다.
두번째 항의 $W_i$는 standard Wiener process구요.

### 신뢰도의 Langevin equation 유도

위의 가정에서 방정식을 유도하는 과정은 보충자료[^7]의 Section II에 설명되어 있습니다.
정보 습득이 서로 독립적이기 때문에, 주어진 정보들에 기반한 각 선택지의 조건부 확률은 아래와 같이 적힐 수 있습니다.

[^7]: Supplementary Material [link](https://journals.aps.org/prl/supplemental/10.1103/PhysRevLett.125.218302/Supplement_2.pdf)

\begin{equation}
    P(H^\pm | \xi_{1:n}) = P(H^\pm) \prod_{t=1}^n \frac{P(\xi_t | H^\pm)}{P(\xi_t)}
\end{equation}

따라서 신뢰도는 다음과 같은 점화식을 만족해야합니다.

\begin{equation}
    y_n \equiv \log \qty[\frac{ P(H^+ | \xi_{1:n})}{ P(H^- | \xi_{1:n})}]
    = y_{n-1} + \log \qty[ \frac{P(\xi_n | H^+)}{P(\xi_n | H^-)} ]
\end{equation}

즉, 신뢰도는 확률변수들의 합으로 표현됩니다.
여기서 Donsker's invariance principle[^8]를 이용하면 곧바로 Stochastic process로 근사될 수 있습니다. 
Donsker's theorem는 중심극한정리(Central Limit Theorem)를 확률과정의 경우로 확장한 정리입니다.

[^8]: M.D. Donsker, "An invariant principle for certain probability limit theorems" , Memoirs , 6 , Amer. Math. Soc. (1951) pp. 1–10

보다 상세히 설명하자면, i.i.d이고 평균이 0, 분산이 1인 확률변수열 $ X_1, X_2, \cdots $이 있을 때, 이들의 부분합 $S_n = \sum_{i=1}^n X_i$으로부터 아래와 같은 확률과정 $W^{(n)}(t)$이 $n\rightarrow \infty$의 극한에서 standard Weiner process로 분포의 관점으로 수렴한다는 것이 Donsker's theorem입니다.

\begin{equation}
    W^{(n)}(t) = \frac{S_{\lfloor n t \rfloor }}{\sqrt{n}}, \quad t \in [0,1]
\end{equation}

다시 본문으로 돌아와서, 위에서 정의한 신뢰도의 수열 $y_n$으로부터 연속적인 시간에 따라 주어지는 확률과정 $y(t)$를 유도해봅시다.
여기서 가장 중요한 것은 정보가 연속적으로 제공된다는 가정입니다.
우리는 이 가정 덕분에 정보제공의 주기를 0으로 보내는 극한을 사용할 수 있습니다.
신뢰도를 계산할 시간 구간을 $[0,T]$로 잡고, 정보제공의 주기가 $\Delta t = T / N$이라 가정해봅시다.
시간의 주기가 길어짐에 따라 제공되는 정보의 양이 비례해 증가할 것이기 때문에, 그 과정에서의 신뢰도 변화 역시 시간에 비례하는 평균과 분산을 가지는 확률변수로 주어질 것입니다.
수학적인 설명을 위해 1회 정보제공에 의한 신뢰도 변화를 $X_n^{(N)}$이라 합시다. 

\begin{equation}
    X_n^{(N)} \equiv \log\qty[ \frac{P(\xi_n^{(N)} | H^+)}{P(\xi_n^{(N)} | H^-)} ]
    \label{eq : belief increment}
\end{equation}

여기서 시간 주기를 $k$배 잘게 쪼갰다고 생각해봅시다.
그러면 정보 $\xi_n^{(N)}$은 총 k개의 독립적인 정보의 순열로 다시 적히게 될겁니다.

\begin{equation}
    \xi_n^{(N)} = \xi_{k n}^{(k N)} \wedge \cdots \wedge \xi_{k (n + 1) - 1}^{(k N)}
\end{equation}

이를 Eq.(\ref{eq : belief increment})에 대입해 다시 정리하면 아래와 같은 식을 얻을 수 있습니다.
\begin{equation}
    X_n^{(N)} 
    = \sum_{i=0}^{k - 1} \log \qty[ \frac{P(\xi_{kn +i}^{(k N)} | H^+)}{P(\xi_{kn+i}^{(k N)} | H^-)} ] 
    = \sum_{i=0}^{k-1} X_{kn +i}^{(k N)}
\end{equation}
모든 정보제공이 i.i.d라 정의했으므로 확률변수 $X_n^{(N)}$의 평균과 분산은 조각의 개수 $N$에 따라 변할 것이고, 이를 각각 $g_N, \rho_N^2$라 정의합시다.
\begin{equation}
    g_N = \mathbb{E}\qty[X_n^{(N)}], \quad
    \rho_N^2 = Var\qty[X_n^{(N)}]
\end{equation}
앞에서 구한 확률변수간의 관계식으로부터 평균과 분산이 만족해야할 관계식을 구할 수 있을텐데요.
\begin{equation}
    g_N = k g_{k N}, \quad \rho_N^2 = k \rho_{k N}^2
\end{equation}
이는 평균과 분산이 각각 $N$에 반비례하단 의미, 시간주기 $\Delta t$에 비례한단 의미입니다.
따라서 우리는 그 비례상수를 $g, \rho^2$이라 정의해 사용합시다.
\begin{equation}
    g = \frac{g_N}{\Delta t} = \frac{N g_N}{T}, \quad 
    \rho^2 = \frac{\rho_N^2}{\Delta t} =  \frac{N \rho_N^2}{T} 
\end{equation}

Donker's theorem은 평균 0, 분산 1인 확률변수들의 합에 대한 정리이기 때문에, 평균 0, 분산 1인 확률변수 $\tilde{X}_n^{(N)}$를 정의해 $X_n^{(N)}$을 정리해둡시다.
\begin{equation}
    X_n^{(N)} = g_N + \rho_N \tilde{X}_n^{(N)}
\end{equation}

그러면 신뢰도 $y_n^{(N)}$은 아래와 같이 쓸 수 있게 됩니다.
\begin{equation}
    y_n^{(N)} = \log\qty[\frac{P(H^+)}{P(H^-)}] + n g_N + \rho_N \sum_{k=1}^n \tilde{X}_n^{(N)}
\end{equation}

이걸 연속적인 확률과정으로 극한을 보내기 위해 이산적인 시간에 대해 정의한 신뢰도 순열을 그대로 시각 t에서의 신뢰도로 확장해줍시다.
\begin{equation}
    y(t) 
    = \lim_{N \rightarrow \infty} y_{\lfloor t / \Delta t \rfloor }^{(N)}
    = \lim_{N \rightarrow \infty} y_{\lfloor N t / T \rfloor }^{(N)}
    \equiv \lim_{N \rightarrow \infty} y_{\lfloor N t' \rfloor }^{(N)}, \quad
    t' \equiv \frac{t}{T} \in [0, 1]
\end{equation}

이때 $y_{\lfloor N t' \rfloor }^{(N)}$을 정리해보면,

\begin{equation}
    y_{\lfloor N t' \rfloor }^{(N)}
    = \log\qty[\frac{P(H^+)}{P(H^-)}] + \lfloor N t' \rfloor g_N + \rho_N S_{\lfloor N t' \rfloor }^{(N)},
    \quad
    S_n^{(N)} = \sum_{k=1}^n \tilde{X}_{n}^{(N)}
\end{equation}

와 같은 결과가 나오고, 여기서 비로소 Donsker's theorem을 이용하면 부분합 $S_{\lfloor N t' \rfloor }^{(N)}$이 Weiner process $\sqrt{N} W(t')$으로 수렴한단 사실을 알 수 있습니다.
$N$이 무한대로 가는 극한에서 숫자 버림은 큰 의미가 없을 것이므로 위의 식은 아래와 같이 근사될 수 있습니다.
\begin{align}
    y_{\lfloor N t' \rfloor }^{(N)}
    &\simeq \log\qty[\frac{P(H^+)}{P(H^-)}] +  N t' g_N + \rho_N \sqrt{N} W(t') \newline
    &= \log\qty[\frac{P(H^+)}{P(H^-)}] + g t + \rho W(t)
\end{align}

따라서 연속 확률과정 $y(t)$는 분포의 관점에서 아래의 결과에 수렴합니다.
\begin{equation}
    y(t) \rightarrow \log\qty[\frac{P(H^+)}{P(H^-)}] + g t + \rho W(t)
\end{equation}

이 증명에서 가장 중요한 파트는 Donsker's theorem을 쓸 수 있도록 하는 연속적인 정보제공 가정입니다. 
따라서 이 논문의 전반에서 **순간적으로 제공되는 정보는 전혀 고려하지 않습니다.**

### Drift term의 의미

위 증명에서 한 가지 더 짚을 수 있는 부분은 바로 drift term의 의미에 해당하는 부분입니다.
Drift 항의 계수 $g$는 정의상 아래와 같은 관계식을 만족할텐데요.
\begin{equation}
    g \Delta t = \int P(\xi | H^\pm) \log\qty[\frac{P(\xi | H^+)}{P(\xi | H^-)}] \dd \xi
\end{equation}
우선 그 drift의 방향은 실제로 참인 선택지가 $H^+$인지, $H^-$인지에 따라 달라진단 사실을 확인할 수 있습니다. (Eq.(\ref{eq : langevin equation of belief})에서 부호가 달라지는 이유)
그리고 또 이 값은 $P(\xi | H^+)$와 $P(\xi | H^-)$의 Kullback-Leibler divergence를 의미하기도 합니다.
즉, 선택지에 따라 주어지는 정보가 크게 달라질수록 drift가 크게 나오는 것이죠.
신뢰도의 정의는 model에 이러한 성격을 담기 위해 정의했다고 볼 수 있습니다.






## References
