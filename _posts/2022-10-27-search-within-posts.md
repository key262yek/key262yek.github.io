---
title: "Search within posts in mmistakes-jekyll"
date: 2022-10-26
categories:
  - Blog
tags:
  - Jekyll
  - MMistakes
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

## Introduction
게시글 숫자가 늘어나면서 특정 게시물을 찾아볼 때 검색기능이 필요하단 생각을 하게 되었다. 
처음에는 Jekyll에 수작업으로 검색 기능을 구현하는 게시글들을 찾아보았는데,
우연히 `_layouts/search.html`이 있는 것을 보고 왠지 [mmistakes](https://mmistakes.github.io/minimal-mistakes/docs/configuration/#site-search)에 알아서 검색 기능이 구현되어 있는 것이 아닐까 생각하게 되었다.

확인해보니 mmistakes에는 site-search 기능이 있었고, 설정 방법 역시 매우 간단했다. 

## Turn-on site-search
이건 구현한다기보단 말그대로 `turn-on`에 가까운 방법이다. 
`_config.yml`에 보면 `search : ` 부분이 있을텐데, 이를 `true`로 바꿔주기만 하면 된다.
```yml
search : true
```

## Result
그리고 확인해보면 아래와 같은 검색 페이지가 만들어지는 것을 확인할 수 있다. 
![MMistakes-search](https://key262yek.github.io/assets/images/mmistakes-search.PNG)
