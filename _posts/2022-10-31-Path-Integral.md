---
title: "Book Review : Path Integrals in Physics Thread"
date: 2022-10-31
categories:
  - Review
  - Physics
tags:
  - Path Integral
published: true
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

## Book info
M. Chaichian and A. Demichev, "Path Integrals in Physics", IOP (2001) [link](https://www.routledge.com/Path-Integrals-in-Physics-Volume-I-Stochastic-Processes-and-Quantum-Mechanics/Chaichian-Demichev/p/book/9780367397142?fbclid=IwAR2gKu02koWfBAu2eJGgg4jo4sXHoFHSBMhtbI30P835p8tlynQ26RoiijU)

경로적분은 Feynman의 경로적분으로 유명한 개념이지만, 내게는 통계역학에서 사용되는 Wiener process에서의 경로적분이 더 친숙하다. 
하지만 친숙한 것 치고는 수학적인 이해는 매우 모자란데, 이 책을 정독하면서 공백을 메꿔보고자 한다. 

## Preface
넓은 범주에서 다양한 목적으로 사용되고 있는 경로적분은 그 사용 빈도에 비해 제대로 된 교과서로 정리된 바 없었다. 대부분 강의록의 형태로만 남아있고, 수학적으로 엄밀히 적힌 책의 경우에는 너무 어려워 처음 공부하는 용으로는 적합하지 않다. 그래서 저자들은 책을 저술하며 두 가지에 집중했다. 

첫 번째로 Wiener 식, Feynman 식 경로적분과 위상공간 위에서의 경로적분의 개괄적인 개념을 담고자 하였고,
두 번째로 보다 다양한 독자층을 가정하고 좀더 실용적인 목적으로 서술하고자 하였다.
경로적분은 아직 활발하게 발전하고 있는 방법론이기 때문에 관련한 모든 내용을 담을 수는 없었지만,
전통적인 결과를 주로 담되 최근의 주요한 발견까지 함께 담으려고 최대한 노력하였다. 

각 장은 자체-완결적으로 적혀있기 때문에 독립적인 교과서처럼 읽을 수 있다. 
아직 깊은 이해가 없는 사람들이더라도 1권의 각 장 중 아무데서나 시작할 수 있고, 
깊게 이해하고 있는 사람들은 2권까지 포함해 관심이 가는 곳부터 읽으면 된다. 

## Introduction
경로적분을 처음 사용한 사람은 1920년대에 확산과 브라운 운동을 연구한 수학자 Norbert Wiener였다. 
그 후 Feynman이 1940년대에 양자역학을 재구성하면서 이를 발전시켰다. 
1950년대로 넘어오면서 양자장론 영역에서 많은 연구가 이뤄졌고, 이후에도 상전이, 초유체, 초전도체, Ising 모델, 양자광학, 플라즈마 물리 등의 영역에서 폭넓게 사용되었다.
1955년, Feynman은 polaron 문제를 풀며 양자역학의 variational principle을 발견했고, 이후 양자장론 뿐 아니라 통계역학과 고체역학에서 매우 큰 영향을 주었다.

### Universality
Stochastic process, Quantum mechanics에서의 전이 확률은 아래와 같은 형태를 띈다. 
$$
  W(x_f, t_f | x_0, t_0) \sim \sum_{\textrm{trajectories from } x_0 \textrm{to} x_f} \exp[- \frac{1}{4D} F[x(\tau)]]
$$
또한 통계역학과 양자역학에서의 기댓값은 아래와 같은 형태로 정리된다. 
$$
  \expval{A} \sim \sum_{configurations} A(\varphi) \exp[- \frac{1}{\hbar} S[\varphi]]
$$
통계역학에서는 $\hbar$가 $k_B T$로 바뀌고, Action $S$가 에너지 $E$로 바뀌는 차이일 뿐이다. 
이 경우, 이들의 단순합은 쉽사리 무한대가 나오기 때문에 이를 해소하는과정이 필요하다. 통계역학에서는 이를 두고 열역학적 극한이라 부르고, 양자장론에서는 재규격화라고 한다. 
다만 둘은 규격화 방식에 있어 차이가 있는데, 통계역학적 극한은 시스템 크기를 점차 키워가는 방식이라 파장의 상한을 정해두고 계산하는 방식이고 (Infrared cut-off), 
재규격화는 최소 단위 길이를 정함으로써 파장의 하한을 정해두고 계산하는 방식(Ultraviolet cut-off)이다. 
이러한 차이는 통계역학과 양자장론을 병합하기 어렵게 하는 요소이며, 전통적으로 두 분야가 전혀 다른 영역으로 취급받던 이유다.
이처럼 경로적분은 적용되는 분야가 상이함에도 불구하고 보편적인 형태를 보여준다.

### Differ from multiple finite-dimensional integrals
얼핏 보았을 때 경로적분은 범함수의 적분처럼 보입니다.
범함수의 적분은 Volterra에 의해 1965년 아래롸 같은 규칙으로 사용되었는데,
1. 함수를 쪼개 $N$개 변수의 함수로 범함수를 변환함
2. 적분을 수행함
3. $N$을 무한대로의 극한으로 보냈을 때, 적분 결과의 극한을 구함. 

하지만 경로적분에서는 이러한 방식이 작동하지 않았고, Wiener process를 밑거름 삼아 보다 더 정밀한 경로적분의 정의를 세울 수 있었다.
이를 간단히 살펴보자. 
실수 벡터 $\mathbb{R}^n$는 Lebesgue measure가 존재하여 적분이 가능합니다. 그럼 같은 성질을 가지는 measure가 무한차원에서도 존재해서 무한차원 벡터의 함수로 적분이 가능할까?
그렇지 않다. 증명은 다음과 같다.  
무한차원 공간 $\mathbb{R}^\inf$에서의 orthonormal basis $\left{ e_1, e_2, \cdots \right}$를 생각했을 때,
$e_k$를 중심으로 하는 반지름 $1/2$인 구 $B_k$와, 원점을 중심으로 하는 반지름 $2$인 $B$를 생각할 수 있다.
이 공간에 measure $\mu$가 있다면, $0 < \mu(B_1) = \mu(B_2) = \cdots < \inf$를 만족할 것이고, 
또한 이들은 서로 교집합이 없기 때문에 measure의 $additivity$ 성질에 따라 $\mu(B) > \sum_k B_k =  \inf$가 된다. 
bounded set의 measure가 무한대일 수 없기 때문에 이러한 measure는 존재하지 않는다. 
 결과적으로 유한 차원 벡터의 함수와 무한차원 범함수의 공간은 같은 형태의 measure로 취급될 수 없고, 범함수의 적분은 그 자체에 대한 이론을 새로이 구축할 필요가 있습니다.




## 1. Path integrals in classical theory
[1.1 Brownian motion]()  
[1.2 Wiener path integrals and stochastic processes]()  

## 2. Path integrals in quantum mechanics
[2.1 Feynman path integrals]()  
[2.2 Path integrals in the Hamiltonian formalism]()  
[2.3 Quantization, the operator ordering problem and path integrals]()  
[2.4 Path integrals and quantization in spaces with topological constraints]()  
[2.5 Path integrals in curved spaces, spacetime transformations and the Coulomb problem]()  
[2.6 Path integrals over anticommuting variables for fermions and generalizations]()  

## 3. Quantum field theory
[3.1 Path-integral formulation of the simplest quantum field theories]()  
[3.2 Path-integral quantization of gauge-field theories]()  
[3.3 Non-perturbative methods for the analysis of quantum field models in the path-integral approach]()  
[3.4 Path integrals in the theory of gravitation, cosmology and string theory: advanced applications of path integrals]()  

## 4. Path integrals in statistical physics
[4.1 Basic concepts of statistical physics]()  
[4.2 Path integrals in classical statistical mechanics]()  
[4.3 Path integrals for indistinguishable particles in quantum mechanics]()  
[4.4 Field theory at non-zero temperature]()  
[4.5 Superfluidity, superconductivity, non-equilibrium quantum statistics and the path-integral technique]()  
[4.6 Non-equilibrium statistical physics in the path-integral formalism and stochastic quantization]()  
[4.7 Path-integral formalism and lattice systems]()  

