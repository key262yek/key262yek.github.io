---
title: "Post의 read-time을 update date로 바꾸기"
date: 2021-03-11T16:34:30-04:00
categories:
  - Blog
tags:
  - HTML
  - Github Blog
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

현재 블로그의 게시글 옆에는 읽는데 걸리는 예상 시간(read-time)이 있습니다.
별로 쓸모있는 정보는 아니기에, 이를 post의 업데이트된 시각으로 바꿔쓸 수 있도록 하고 싶었는데요.
옛날 자료들을 찾아보면 plugin을 통해서 업데이트 시각을 계산하도록 하고 있는데, 2021년 release된 Minimal-Mistakes 템플릿을 기준으로 했을 때는 업데이트 시각은 자체적으로 다른 plugin 없이 곧바로 사용할 수 있습니다. 
당연한것이, post의 각주부분을 보시면 태그, 카테고리와 함께 업데이트 시간이 있으니 자체적으로 계산하고 있단 것을 확인할 수 있습니다.

### 업데이트 시각 띄우기

원하는 위치에 시각을 띄우기 전에 우선 해당 정보를 어떻게 확인할 수 있는지를 찾아보았습니다.
만들어진 페이지의 html 소스를 확인해보면 `page_meta` 클래스 아래에 `page_date` 클래스 정보가 담겨져 있는걸 확인할 수 있는데요.
`page_meta`는 *_layouts/single.html*에서 찾을 수 있었습니다.
*page__date.html*를 불러와 아이콘부터 업데이트 시각까지 한번에 출력하는 방법입니다.

{% include page__date.html %}

그에 반해, Post 제목 아래에 있는 read-time은 *page__meta.html*을 불러오는데, 여기에서 `page__meta-readtime` 클래스가 바로 예상시간을 출력해주는 부분입니다.
readtime을 계산하는 부분을 주석처리하고, post__date.html를 삽입하면 해결됩니다.
각각의 원문 파일은 *_includes* 폴더에 있습니다.



