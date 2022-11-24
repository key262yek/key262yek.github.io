---
title: "Interactive plot blog posting"
date: 2022-11-20
categories:
  - Programming
  - Blog
tags:
  - Plotly
  - Kramdown
  - Jekyll
  - mmistakes
published: true
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

논문을 정리하는 과정에서 여러 이미지를 임의로 그려서 올려봤는데, 썩 맘에 들지 않았다.
관련해서 직접 도표를 뽑아 업로드 하는 방법을 생각했는데, 문제는 이 도표가 3차원 그림이라 한 눈에 딱 잘 들어오는 각도를 고르기 어렵다는 점이었다.
그래서 반응형 도표를 만들어 이리저리 돌려볼 수 있도록 만들고자 했다.
이 글에서는 Plotly 도표 html을 만들고 블로그에 업로드 하는 방법을 기록하고자 한다.

## Plotly 도표 만들기
아래와 같은 코드로 Plotly 도표를 만들어내고, html 파일로 export 할 수 있다. 
``` python
import numpy as np
import plotly.express as px
import plotly.graph_objects as go 

# Creating dataset
N = 2000
x = np.random.uniform(low=-3.0, high=3.0, size=N)
y = np.random.uniform(low=-3.0, high=3.0, size=N)
z = np.cos(0.2 * x) + np.sin(0.1 * y) 

# Interactive plot with plotly
data = go.Scatter3d(
    x = x, 
    y = y, 
    z = z, 
    mode = 'markers', 
    marker = dict(
        size = 0.7, 
        color = z, 
        colorscale='Viridis', 
        opacity = 0.8
    )
)
fig = go.Figure(data = data)

fig.update_layout(margin = dict(l=0, r=0, b=0, t=0))
fig.write_html("test.html")
fig.show()
```

## Upload html
결과로 나온 `test.html`을 `_include`에 올린 후(제 경우에는 `_include/plots`라는 폴더를 새로 만들어 파일을 담았다) 아래와 같은 코드를 markdown 파일에 추가하면 블로그에 반응형 도표를 띄울 수 있게 된다.
```html
{% raw %} {% include plots/test.html %} {% endraw %}
```

{% include plots/real_data.html %}