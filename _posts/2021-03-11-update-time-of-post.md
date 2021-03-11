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

지금은 *_config.yml*의 일부만 수정해주면 됩니다.
post의 기본 양식을 결정하는 defaults 부분에서 read_time을 false로 바꿔주고, show_date 부분을 추가해주면 됩니다.

~~~ yaml
# Defaults
defaults:
  # _posts
  - values:
      read_time: false
      show_date: true
~~~

이렇게 하더라도 post 부분에는 날짜만 나오고, 해당 날짜가 작성일인지 업데이트된 날짜인지 표기가 나오지 않습니다.
이를 위해서 *_includes/page__meta.html*을 일부 수정해줍니다.
아이콘을 출력하는 부분과 time을 출력하는 부분 사이에 아래의 문장을 (앞부분 중괄호 사이 띄어쓰기는 없앤채로) 넣어주면 됩니다.

~~~ html
{ { site.data.ui-text[site.locale].date_label | default: "Updated:" }} 
~~~

자세한 사항은 이 [commit](https://github.com/key262yek/key262yek.github.io/commit/c955e2bf588b2dd22da68c18d13a6f31e1aeea1e)을 통해 확인하세요. 


