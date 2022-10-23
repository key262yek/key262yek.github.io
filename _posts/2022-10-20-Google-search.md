---
title: "Make the blog searchable by Google"
date: 2022-10-20
categories:
  - Blog
tags:
  - Github Blog
  - Google Search
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

## Introduction
Github 블로그의 결과물은 가만히 둔다고 검색되지 않습니다. 
검색시 site 태그로 블로그 주소를 고정하면 아예 검색결과가 없다고 나오구요.
구글에서 검색되도록 하기 위해서는 [Google search console]("https://search.google.com/search-console/welcome?hl=ko")에 블로그를 등록해야만 합니다. 
이 글에서는 Github 블로그를 Search console에 등록하는 방법에 대해 기술합니다.

## Domain? URL?
<img src="https://key262yek.github.io/assets/images/google_search_console.PNG" alt="Goggle search console" width="400"/>

Google search console에 접속하면 위와 같은 창이 뜹니다. 
우리는 DNS로부터 도메인을 사서 쓰는 것이 아니기 때문에 오른쪽 방법을 사용해야만 합니다.
오른쪽 방법을 사용할 때는 아래와 같은 방법을 사용할 수 있습니다. 

- HTML 파일 : 웹사이트에 HTML 파일 업로드 (권장)
- HTML 태그 : 사이트 홈페이지에 메타태그 추가
- Google 애널리틱스 : Google 애널리틱스 계정 사용
- Google 태그 관리자 : Google 태그 관리자 계정 사용
- 도메일 이름 공급업체 ; DNS 레코드와 Google 연결
 
구글에서는 html 파일을 받아 업로드하는 것을 권장하는데,
Jekyll을 사용하는 방법은 보다 더 간단한 방법도 존재합니다. 
두 방법을 다 알아봅시다. 

### HTML File upload
먼저 HTML 파일을 이용하는 방법을 써봅시다. 
사이트 경로를 적어 확인 버튼을 눌렀을 때 제공된 `google********.html`과 같은 파일을 `_config.yml`파일이 있는 경로에 업로드해주면 됩니다. 

### Jekyll config
스크롤을 내리다보면 아래에 여러가지 다른 방법을 알려주게 되는데,
그 중에 HTML 태그를 선택해 나오는 meta tag의 content 부분을 `_config.yml`의 아래 부분에 삽입해줍니다.
```yml
# SEO Related
google_site_verification :
```

## Sitemap.yml and robots.txt
이 부분은 이미 jekyll에서 자동으로 만들어주는 plugin이 있습니다. 
하지만 이 기능은 Github pages에서는 활용할 수 없습니다. 
```yml
# Plugins (previously gems:)
plugins:
  - jekyll-paginate
  - jekyll-sitemap
  - jekyll-gist
  - jekyll-feed
  - jekyll-include-cache
```

따라서 우리는 아래와 같은 파일을 직접 root 디렉토리에 넣으면 됩니다. 
이때 sitemap.xml의 위치와 이름을 정확하게 robots.txt에 적는 것이 중요합니다. 
```xml
{% raw %}---{% endraw %}
{% raw %}layout: null{% endraw %}
{% raw %}---{% endraw %}

{% raw %}<?xml version="1.0" encoding="UTF-8"?>{% endraw %}
{% raw %}<urlset xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://www.sitemaps.org/schemas/sitemap/0.9 http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd"
                xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">{% endraw %}
{% raw %}    {% for post in site.posts %}{% endraw %}
{% raw %}    <url>{% endraw %}
{% raw %}        <loc>{{ site.url }}{{ post.url }}</loc>{% endraw %}
{% raw %}        {% if post.lastmod == null %}{% endraw %}
{% raw %}        <lastmod>{{ post.date | date_to_xmlschema }}</lastmod>{% endraw %}
{% raw %}        {% else %}{% endraw %}
{% raw %}        <lastmod>{{ post.lastmod | date_to_xmlschema }}</lastmod>{% endraw %}
{% raw %}        {% endif %}{% endraw %}

{% raw %}        {% if post.sitemap.changefreq == null %}{% endraw %}
{% raw %}        <changefreq>weekly</changefreq>{% endraw %}
{% raw %}        {% else %}{% endraw %}
{% raw %}        <changefreq>{{ post.sitemap.changefreq }}</changefreq>{% endraw %}
{% raw %}        {% endif %}{% endraw %}

{% raw %}        {% if post.sitemap.priority == null %}{% endraw %}
{% raw %}        <priority>0.5</priority>{% endraw %}
{% raw %}        {% else %}{% endraw %}
{% raw %}        <priority>{{ post.sitemap.priority }}</priority>{% endraw %}
{% raw %}        {% endif %}{% endraw %}

{% raw %}    </url>{% endraw %}
{% raw %}    {% endfor %}{% endraw %}
{% raw %}</urlset>{% endraw %}
```

```
User-agent: *
Allow: /

Sitemap: https://key262yek.github.io/sitemap.yml
```

## register sitemap
이제 해당 sitemap 파일을 google search console에 등록해주면 됩니다. 

<img src="https://key262yek.github.io/assets/images/add_sitemap.PNG" alt="Sitemap" width="400"/>

## Result
바로 사이트 게시물을 검색할 수는 없습니다. 빠르면 일주일, 느리면 한 달까지도 걸리는 것 같은데 기다리고 나면 아래와 같이 검색이 가능합니다. 

<img src="https://key262yek.github.io/assets/images/search_result.png" alt="Search Result" width="400"/>
