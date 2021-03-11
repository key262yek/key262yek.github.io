---
title: "Kramdown 사용법"
date: 2021-03-09T15:08:30-04:00
categories:
  - Blog
tags:
  - Markdown
  - Github Blog
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

Markdown은 자체 표기법을 사용해 HTML과 같은 웹페이지 양식을 만드는 기능을 지칭합니다.
지금 작성하고 있는 블로그의 글들은 모두 확장자 .md로 쓰여있는데, 이는 markdown의 약자로 문법을 잘 지켜 작성한다면 블로그의 글에 여러 효과를 쉽게 줄 수 있습니다.
Markdown은 John Gruber와 Aaron Swartz가 2004년 처음 만든 이래로 표준화되지 못하고 PHP MArkdown Extra, Multimarkdown, Github Flavored Markdown 등으로 파편화되었는데요.
이 중 Github 블로그는 Kramdown이라는 Markdown을 지원합니다. 
앞으로 원활하고 깔끔한 블로그 글 작성을 위하여 Kramdown의 기능을 익히는 포스트를 첫번째 포스트로 작성해볼까 합니다.

----------------------------

### 1. 헤더

헤더 문장을 작성하는 경우에 Kramdown에서는 총 6개 양식의 헤더를 작성할 수 있습니다.
헤더 문장 앞에 #을 갯수에 맞게 붙여주고 아래로 한 줄을 띄워주면 됩니다.


~~~~~~ markdown
# H1 Header

## H2 Header

### H3 Header

#### H4 Header

##### H5 Header

###### H6 Header
~~~~~~


> # H1 Header
> {: .no_toc}
>
> ## H2 Header
>{: .no_toc}
> 
> ### H3 Header
> {: .no_toc}
> 
> #### H4 Header
> {: .no_toc}
> 
> ##### H5 Header
> {: .no_toc}
> 
> ###### H6 Header
> {: .no_toc}

------------

### 2. Inline Code

문장 사이사이에서 코드를 삽입하고 싶을 때, 특정 부분을 강조하고 싶을 때, 혹은 syntax character라 그냥 쓰면 안되는 경우 (이 경우는 backslash를 이용해 escape할 수도 있는 것 같다) inline code block을 이용하면 좋습니다.
사용 방법은 \`를 이용해 내용을 감싸주면 됩니다.

~~~ markdown
`이렇게 강조할 수 있어요`
~~~


> `이렇게 강조할 수 있어요`  
> `$ \ ~~~` 이렇게 syntax character를 쓸 수도 있어요. 


---

### 3. Code Block

코드 블럭은 작성한 Markdown 원문을 HTML 문법으로 바꾸지 않고 그대로 보여주고 싶을때나, 프로그래밍 언어에 따라 하이라이트를 주고자 할 때 사용할 수 있습니다.
기본적으로 코드 블럭은 아래와 같은 두 가지 방법으로 쓸 수 있습니다. 
첫번째 방법은 4칸을 들여쓰는 방법입니다.


~~~~~~ markdown
    이렇게 코드 블럭을
    쓸 수 있습니다.
~~~~~~


    이렇게 코드 블럭을
    쓸 수 있습니다.

두 번째 방법은 물결표시 \~로 Code를 감싸는 것입니다.
이때 중요한 것은 코드 블럭을 닫는 부분의 물결표가 시작할 때보다 많아야합니다.

~~~~~~~ markdown
~~~~
이렇게 코드 블럭을
쓸 수 있습니다.
~~~
이렇게 개수가 적으면 코드 블럭이
끝나지 않습니다.
~~~~
~~~~~~~

~~~~ markdown
이렇게 코드 블럭을
쓸 수 있습니다.
~~~
이렇게 개수가 적으면 코드 블럭이
끝나지 않습니다.
~~~~

프로그램 코드를 코드 블럭으로 감쌀 때는 가독성을 위해 syntax highlight를 넣곤 합니다.
Kramdown에서 프로그램 코드의 하이라이트를 넣기 위해서는 코드 블럭이 시작하는 ~~~ 옆에 언어의 이름을 기입하면 됩니다.


~~~~~ markdown
~~~ ruby
def what?
 42
end
~~~

~~~ java
public static void main(String[] args){
}
~~~
~~~~~


~~~ ruby
def what?
 42
end
~~~

~~~ java
public static void main(String[] args){
}
~~~

----------------------------


### 4. 가로선

글을 작성하다보면 내용 구분이 필요할 때 가로선을 쓸 때가 있죠.
가로선은 아래와 같은 방법들로 그릴 수 있습니다.


~~~~~ markdown
* * *

---

  _  _  _  _

---------------
~~~~~


> * * *
>
> ---
>
>  _  _  _  _
>
> ---------------

----------------------------

### 5. 개행

원문에 그냥 줄넘김해서 쓴다해도 결과물에는 개행이 나타나지 않습니다.
개행을 위해서는 공백을 두 개 작성하면 됩니다.

~~~ markdown
This is paragraph  
which contains a hard line break
~~~

> This is paragraph    
> which contains a hard line break

----------------------------


### 6. 강조

이텔릭체와 진한 글씨 등의 강조구문은 두 가지 방법으로 작성할 수 있습니다.

~~~~ markdown
이것은 *이탤릭*, **진하게**  
이것도 _이탤릭_, __진하게__
~~~~


> 이것은 *이탤릭*, **진하게**  
> 이것도 _이탤릭_, __진하게__

----------------------------

### 7. 블록 인용

한 줄 혹은 한 단락을 직접 인용하고자 할 때, \> 문자를 맨 앞에 두어 강조할 수 있습니다.
여러 번 사용해 다단계 블록 인용도 가능하구요.

~~~ markdown
> 첫번째 블록 인용
> > 두번째 블록 인용
> ### Header도 됩니다
~~~

> 첫번째 블록 인용
> > 두번째 블록 인용    
> 
> ### Header도 됩니다 
> {: .no_toc}
> 

-------

### 8. 링크

대괄호 \[ 안에 화면에 보여질 링크명을 적고, 괄호 \( 안에 링크 주소를 넣어 사용할 수 있습니다.
링크 주소 뒤에 한 칸을 띄어 title 속성도 줄 수 있습니다.

~~~~ markdown
A [링크](http://kramdown.gettalong.org)

A [링크](http://kramdown.gettalong.org "hp")

A [링크][kramdown hp]
to the homepage.

[kramdown hp]: http://kramdown.gettalong.org "hp"


A link to the [kramdown hp].

[kramdown hp]: http://kramdown.gettalong.org "hp"
~~~~

> A [링크](http://kramdown.gettalong.org)
>
> A [링크](http://kramdown.gettalong.org "hp")
>
> A [링크][kramdown hp]
> to the homepage.
>
> [kramdown hp]: http://kramdown.gettalong.org "hp"
>
>
> A link to the [kramdown hp].
>
> [kramdown hp]: http://kramdown.gettalong.org "hp"

-------

### 9. 이미지

깃헙 저장소의 이미지 소스 주소를 연결함으로써 이미지를 첨부할 수 있습니다.
이미지는 링크와 비슷한 방식이지만 대괄호 앞에 느낌표를 붙여줍니다.

~~~~ markdown
이미지 : ![mine](https://key262yek.github.io/assets/images/new_bio_photo.jpg)
~~~~

> 이미지 : ![mine](https://key262yek.github.io/assets/images/new_bio_photo.jpg)

------

### 10. 각주

특정 문자를 지수 부분으로 올려주는 방법으로 각주를 표기할 수 있습니다.

~~~ markdown
각주1 선언부분 a footnote[^1].  
각주2 선언부분 a footnote[^2].

[^1]: 각주에 대한 설명 내용 부분 (문서 최하단)
[^2]: 각주에 대한 설명 내용 부분 (문서 최하단)
~~~

> 각주1 선언부분 a footnote[^1].  
> 각주2 선언부분 a footnote[^2].

[^1]: 각주에 대한 설명 내용 부분 (문서 최하단)
[^2]: 각주에 대한 설명 내용 부분 (문서 최하단)

--- 

### 11. 문단

문단은 아래와 같은 문법을 따라 구분할 수 있습니다.

1. 개행을 하더라도 한줄로 같은 문단으로 묶임
2. 띄어쓰기 두 번을 붙이면 같은 문단에 묶이지만 개행이 되어 보인다.
3. 한 줄 더 개행하면 서로 다른 문단에 묶임.

~~~ markdown
같은 라인 문단 1, 
같은 라인 문단 2

같은 문단 다른라인 1  
같은 문단 다른 라인 2
~~~

> 같은 라인 문단 1, 
> 같은 라인 문단 2
> 
> 같은 문단 다른라인 1  
> 같은 문단 다른라인 2 \\
> 같은 문단 다른 라인 3

### 12. Ordered List

1,2,3 순번이 있는 리스트를 지원합니다. 
순번별로 개행해주어야 꼬이지 않습니다.

~~~ markdown
순서 리스트 1  

1. 리스트 Item 1  
2. 리스트 Item 2  
2. 2번째인듯한 리스트 Item 3
   , 다음줄인듯한 3번째 뒷부분 
~~~

> 순서 리스트 1  
> 
> 1. 리스트 Item 1  
> 2. 리스트 Item 2  
> 2. 2번째인듯한 리스트 Item 3
>    , 다음줄인듯한 3번째 뒷부분

~~~ markdown
순서 리스트2

1. 리스트 Item 1
    > 블록 인용과 함께
    
    # 헤더와 함께

2. 리스트 Item 2
~~~

> 순서 리스트2
> 
> 1. 리스트 Item 1
>     > 블록 인용과 함께
>     
>     # 헤더와 함께
> 
> 2. 리스트 Item 2

~~~ markdown
순서 리스트 3

1. 리스트 Item 1
    1. 서브 리스트 Item 1
    2. 서브 리스트 Item 2
    3. 서브 리스트 Item 3
2. 리스트 Item 2
~~~


> 순서 리스트 3
> 
> 1. 리스트 Item 1
>     1. 서브 리스트 Item 1
>     2. 서브 리스트 Item 2
>     3. 서브 리스트 Item 3
> 2. 리스트 Item 2

----

### 13. Unordered List

순번 없는 리스트를 지원합니다.

~~~ markdown
순서 없는 리스트1

* Item1
,Item1의 뒷부분
~~~

> 순서 없는 리스트 1
> 
> * Item 1
> , Item 1의 뒷부분
> * Item 2

~~~ markdown
순서 없는 리스트2

* Item 1
+ Item 2
- Item 3
~~~


> 순서 없는 리스트 2
> * Item 1
> + Item 2
> - Item 3


---

### 14. 정의 리스트 

~~~ markdown
용어1
: 정의1
: 정의2

용어2
용어3(2번 같은 뜻)
: 용어2,3번의 정의

용어4

: 정의 문단 1
: 정의 문단 2
~~~

> 용어 1
> : 정의 1
> : 정의 2
> 
> 용어 2
> 용어 3 (2번과 같은 뜻)
> : 용어 2, 3의 정의
> 
> 용어 4
> 
> : 정의 문단 1
> : 정의 문단 2

---

### 15. 테이블

테이블은 아래와 같은 문자들로 구분선을 그어줄 수 있습니다.

* | :파이프 라인으로 컬럼을 구분
* :----- : 헤더와 body를 구분해줌.
* --- : 테이블 body끼리를 구분
* === : 테이블 foot을 구분

~~~ markdown
| 헤더1 | 헤더2 | 헤더3 |
|:--------|:-------:|--------:|
| 컬럼1   | 컬럼2   | 컬럼3   |
| 컬럼4   | 컬럼5   | 컬럼6   |
|----
| 컬럼1   | 컬럼2   | 컬럼3   |
| 컬럼4   | 컬럼5   | 컬럼6   |
|=====
| Foot1   | Foot2   | Foot3
{: rules="groups"}
~~~

> | 헤더1 | 헤더2 | 헤더3 |
> |:--------|:-------:|--------:|
> | 컬럼1   | 컬럼2   | 컬럼3   |
> | 컬럼4   | 컬럼5   | 컬럼6   |
> |----
> | 컬럼1   | 컬럼2   | 컬럼3   |
> | 컬럼4   | 컬럼5   | 컬럼6   |
> |=====
> | Foot1   | Foot2   | Foot3
> {: rules="groups"}





