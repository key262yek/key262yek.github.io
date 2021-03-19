---
title: "MathJax v3로 업그레이드하기, 기능 추가하기"
date: 2021-03-19T16:36:30-04:00
categories:
  - Blog
tags:
  - Jekyll
  - MathJax
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

[지난 번]({% post_url 2021-03-10-mathjax-to-mmistakes %})에는 Jekyll blog에 MathJax를 연결해 수식을 입력할 수 있도록 했습니다.
이때의 코드를 확인해보면 연결한 MathJax의 버젼이 2.7.5임을 알 수 있습니다. 

~~~ javascript
<script type="text/javascript" async
    src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-MML-AM_CHTML">
</script>
~~~

이 상태로도 사용성은 매우 좋았지만 몇 가지 문제가 있어 최신이면서 안정적인 3.0 버젼으로 업그레이드를 하고자 합니다.
이유는 크게 두 가지입니다.

* LaTex의 physics 패키지 제공
* 여러 질답들이 v3를 기준으로 작성된 경우가 많음

physics 패키지는 여러 물리 기호들을 작성하는데 매우 많은 도움을 주는 패키지라 자주 사용하곤 하는데요.
평소 습관대로 MathJax 구문을 작성하다보면 physics 패키지의 기능들이 없는게 많이 아쉬웠습니다.
찾아보니 MathJax에서도 LaTex extension을 몇 가지 제공하고 있는데, v2에서는 physics가 없고 v3에서만 제공되고 있더군요.

그리고 equation numbering 등의 기능을 추가하는 방법들을 검색하는 과정에서도, 많은 답변들이 v3를 기준으로 설명하고 있어서 문법 차이 때문에 혼란이 컸습니다.
어차피 physics 패키지를 이용해야하기도 하고, 어차피 다들 v3로 넘어간 것으로 보이니 저도 업그레이드를 해보겠습니다.

### Loading MathJax

가장 먼저 수정해줘야할 부분은 MathJax를 불러오는 부분의 스크립트입니다.
기존에는 2.7.5 버젼을 불러왔으나, 지금은 3.0.0을 불러올겁니다.
항상 최신 버젼을 불러오도록 할 수도 있으나([참고](http://docs.mathjax.org/en/latest/web/start.html)) 문법 변화나 에러에 취약하게 될 수 있습니다.

~~~ javascript
... scripts.html 중에서

<script id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3.0.0/es5/tex-mml-chtml.js">
</script>
~~~

### Configuring MathJax

설정 변경 코드는 v3로 넘어오면서 매우 간단해졌습니다.
~~~ javascript
... scripts.html 중에서

<script>
MathJax = {
  loader:  {
    load: ['[tex]/physics']                    // physics 패키지를 lodaing해줄 loader
  },
  tex: {
    inlineMath: [['$', '$'], ['\\(', '\\)']],  // 어떻게 math mode를 시작할 것인지
    packages: {'[+]': ['physics']},            // 추가할 package
    tags: 'ams'                                // auto-numbering
  }
};
</script>
~~~

### Enjoy MathJax v3
이런 과정을 거치면 이제 physics 패키지를 지원하는 MathJax v3를 이용할 수 있습니다.
다만 auto-numbering을 위해서는 `$`기호가 아니라 실제 LaTex의 numbered equation 문법에 맞춰 equation, align 등의 모드를 열어 작성해줘야합니다.


\begin{equation}
    \bra{\psi} = \sum_{\omega} \bra{\omega} \braket{\omega}{\psi}
\end{equation}


