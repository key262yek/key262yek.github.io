---
title: "Review: Deep Unsupervised Learning using Nonequilibrium Thermodynamics"
date: 2022-10-26
categories:
  - Review
tags:
  - Deep Learning
  - Statistical_Physics
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

## Paper information
J.Sohl-Dickstein et al, "Deep Unsupervised Learning using Nonequilibrium Thermodynamics", 2015 ([link](https://arxiv.org/pdf/1503.03585.pdf))

## Abstract
A central problem in machine learning involves modeling complex data-sets using highly flexible families of probability distributions in which learning, sampling, inference, and evaluation are still analytically or computationally tractable. Here, we develop an approach that simultaneously achieves both flexibility and tractability. The essential idea, inspired by non-equilibrium statistical physics, is to systematically and slowly destroy structure in a data distribution through an iterative forward diffusion process. We then learn a reverse diffusion process that restores structure in data, yielding a highly flexible and tractable generative model of the data. This approach allows us to rapidly learn, sample from, and evaluate probabilities in deep generative models with thousands of layers or time steps, as well as to compute conditional and posterior probabilities under the learned model. We additionally release an open source reference implementation of the algorithm.

## Model Characteristics
Generative model with Multiscale CNN

## What is generative model
Input data를 기반으로 (없을 수도 있음) 생성물의 확률분포 $p(x;\theta)$를 구하고, 가장 확률이 높은 생성물을 출력해주는 모델. 

### Limitation of other Generative models
<img src="https://key262yek.github.io/assets/images/generative-overview.png" alt="drawing">

- GAN: Potentially unstable training, less diversity due to adversarial training 
- VAE: relies on surrogate loss
- Flow-based - have to use specialized architectures to construct reversible transform.

## Purpose

### Tractable and Flexible
- Tractable : Analytically evaluated and eaily fit
- Flexible : can be molded to fit structure in arbitrary data

**Face tradeoff between two objectives**

Thus we want to make
1. Extremely flexible
2. Exact sampling
3. Easy multipilcation with other distributions
4. log likelihood, probability of individual states to be cheaply evaluated.

## Idea - Analytically tractable reverse process

 <img src="https://key262yek.github.io/assets/images/diffusion_model_forward_process.png" alt="drawing"/>
 <img src="https://key262yek.github.io/assets/images/diffusion_model_reverse_process.png" alt="drawing"/>


Forward process
\begin{equation}
    q(x^{1:T} | x_0) = \prod_{t=1}^T q(x^t | x^{t-1}) = \prod_{t=1}^T N(x^t; \sqrt{1 - \beta_t} x^{t-1}, \beta_t \mathbb{I})
\end{equation}

Reverse process
\begin{equation}
    p(x^{0:T}) = p(x^T) \prod_{t=1}^T p_{\theta} (x^{t-1} | x^t) = p(x^T) \prod_{t=1}^T N(x^{t-1}; f_{\mu}(x^t, t), f_{\Sigma}(x^t, t))
\end{equation}

## Benefit to use
- Not Requiring adversarial training
- Scalability
- Parallelizability

### Trilemma
<img src="https://key262yek.github.io/assets/images/GANs_Diffusion_Autoencoders.png" alt="drawing"/>

## Model characteristics

### Input data
\begin{equation}
  y = (y_1^\mu, y_1^\Sigma, \cdots) \in \mathbb{R}^{2J},\quad J : \textrm{Number of pixels}
\end{equation}

### Train parameter 
\begin{equation}
    \textrm{Gaussian : } f_\mu(x^t, t), \; f_{\Sigma}(x^t, t) \\
    \textrm{Binomial : } f_b(x^t, t) \quad\quad\quad\quad\;
\end{equation}

### Score function
- Log likelihood 
\begin{equation}
    L = \int d x^0 q(x^0) \log p(x^0)
\end{equation}

### Network
<img src="https://key262yek.github.io/assets/images/Sohl-Dickstein(2015).png" alt="drawing"/>
