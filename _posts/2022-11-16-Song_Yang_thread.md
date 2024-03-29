---
title: "Paper Review : Song and Ermon thread"
date: 2022-11-16
categories:
  - Review
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

지난 번 Sohl-Dickstein의 2015년 논문을 정리한 이후, diffusion model에 관한 논문을 계속 읽어보려한다. 
이번 글 타래에서는 그 중에서 diffusion model의 또다른 이름인 Score-based generative modeling을 알리게 한 Yang Song과 Stefano Ermon의 연구를 정리해보려 한다. 
대부분의 내용은 논문을 참고하였고, 또한 저자가 직접 작성한 [블로그](https://yang-song.net/blog/2021/score/)글 역시 함께 참고하였다. 

["Generative Modeling by Estimating Gradients of the Data Distribution", *NeurIPS* (2019)](% post_url 2022-11-16-song_2019 %)   
["Score-based generative modeling through stochastic differential equation", *ICLR* (2020)](% post_url 2022-11-20-song_2020 %)   
["Improved techniques for training score-based generative models", *NeurIPS* (2020)](% post_url 2022-11-20-song_2020_2 %)   