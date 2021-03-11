---
title: "Jekyll theme에 수식 입력기 추가하기"
date: 2021-03-10T15:36:30-04:00
categories:
  - Blog
tags:
  - Jekyll
  - Github Blog
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

아무래도 물리나, 수학을 전공하다 보니 블로그 글에 수식입력기가 빠르게 필요해졌다.
Jekyll theme과 수식입력을 같이 검색하면 많은 예시자료들이 나오는데, 보면 remote theme을 이용하는 경우와 theme git을 모두 clone한 경우를 구분하지 않고 설명하고 있다. 
결과적으로는 구분할 필요가 없긴 하지만, 초보자이면서 remote theme을 이용하고 있는 내 입장에서는 

> '파일이 없는데 지금이라도 파일들을 모두 받아야하나? 받는 과정에서 지금까지 셋팅한게 다 덮이면 어떡하지?'

하는 걱정이 들 수밖에 없었다. 다행히 Jan Meppe의 [블로그](https://www.janmeppe.com/blog/How-to-add-mathjax-to-minimal-mistakes/)에서 remote theme에서 어떤 부분을 고치면 되는지 잘 설명해주었기에, 이를 따라 진행해보았다. (본래 블로그의 Step 1은 여기에 필요 없어 보여서 제외했다.)

### Step 1. scripts.html 복사해오기

우선 설정을 바꾸기 위해 '모든 파일을 복사해와야하는가?'하면 그렇지 않은 것 같다. remote theme의 경우 내 github 저장소에 없는 파일들만 가져오는 방식인 것 같고, 그러니 내가 필요한 파일만 복사해와 수정하면 된다. 블로그 처음 셋팅할 때 폰트랑 폰트 크기 변경한 것도 assets/css/main.scss 파일만 수정했었는데, 같은 원리라 볼 수 있다.

어쨌든 필요한건 [Minimal Mistakes repo](https://github.com/mmistakes/minimal-mistakes)에서 *minimal-mistakes/_includes/scripts.html* 파일을 로컬 저장소의 *_includes/scripts.html*에 복사해오는 것이다. 

### Step 2. scripts.html 수정하기

그 다음 *scripts.html* 파일에 아래의 코드를 추가하면 된다.

~~~~ javascript
<script type="text/javascript" async
    src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-MML-AM_CHTML">
</script>

<script type="text/x-mathjax-config">
   MathJax.Hub.Config({
     extensions: ["tex2jax.js"],
     jax: ["input/TeX", "output/HTML-CSS"],
     tex2jax: {
       inlineMath: [ ['$','$'], ["\\(","\\)"] ],
       displayMath: [ ['$$','$$'], ["\\[","\\]"] ],
       processEscapes: true
     },
     "HTML-CSS": { availableFonts: ["TeX"] }
   });
</script>
~~~~

### Step 3. Latex 문법 사용

이제 Latex 문법을 사용해 Markdown 파일에 수식을 입력할 수 있게 되었다. 문장 사이에 쓰이는 math mode는 `$` 기호나 `\\( \\)`를 이용하면 되고, equation mode를 추가하는 경우에는 `$$`나 `\\[ \\]`로 수식을 감싸주면 된다.

$$
    e^{i \pi} = - 1
$$

### Note. 주의사항

이 방법은 remote theme 상태에서 일부 설정만 바꿔주는 방법이다보니, 만약 theme 파일의 버젼이 크게 달라지게 되면 더이상 작동하지 않을 수 있다. 그 경우 위의 파일을 다시 수정해야만 할 것이다.
