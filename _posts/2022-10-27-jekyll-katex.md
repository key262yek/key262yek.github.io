---
title: "Jekyll에 KaTex 설치하기"
date: 2022-10-27T10:00:00
categories:
  - Blog
tags:
  - Jekyll
  - KaTex
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

## Introduction
지금 블로그에서는 MathJax라는 plugin을 얹어 Tex 문법으로 수식을 작성하고 있다. 
최근 노션을 자주 사용하게 되면서 노션의 수식 입력기는 KaTex를 이용한단 점을 확인했고,
Jekyll에 얹어보려한다. 
물론 기능적으로 MathJax와 큰 차이가 없기 때문에 재미 이상의 효과를 얻을 수는 없을 것이다. 

설치 및 블로그 내용은 [Jekyll-KaTex github](https://github.com/linjer/jekyll-katex)를 참고해 작성하였다. 

## Installation

### Plugin
먼저 Jekyll의 `_config.yml`을 아래와 같이 수정한다. 
``` yml
plugins:
    - jekyll-katex
```

### Gemfile
그 다음 `Gemfile` 에 아래와 같은 plugin block을 추가한다.
``` ruby
group :jekyll_plugins do
  gem 'jekyll-katex'
end
```
그리고 `bundle install` 명령어를 실행한다. 
(ruby가 설치되어 있지 않으면 설치한 후 실행한다)

## Test
그 후 update된 `Gemfile.lock`을 push하면 아래와 같이 수식을 적을 수 있다!

```latex
{% katex %}
c = \pm\sqrt{a^2 + b^2}
{% endkatex %}
```

{% katex %}
c = \pm\sqrt{a^2 + b^2}
{% endkatex %}
