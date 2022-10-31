---
title: "암호화폐 과거 거래 데이터 Rust로 크롤링하기"
date: 2021-04-05T16:36:30-04:00
categories:
  - Programming
tags:
  - Rust
  - Cryptocurrency
  - Crawling
published: false
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

주식, 암호화폐의 수익률이 떨어질 줄을 모르는 시기인지라 남녀노소 주식이나 암호화폐를 건드려보지 않은 사람이 없는 것 같습니다.
나스닥의 90%가 넘는 주식이 모두 200일 평균 성장선보다 위에 있어 대부분의 주식 투자자들이 수익을 보고있고, 3개월 비트코인 성장률이 100%라고 하니 말 다했죠.
이런 엄청난 호황기에도 시드머니가 없는 사람들은 수익을 보더라도 적은 금액에 불과하거나, 레버리지를 과하게 잡아 조금의 하락세에도 큰 손실을 볼 수밖에 없을텐데요.
그래서 지인들 사이에서 종종 나오는 이야기가 있습니다.

> "아, 그냥 복권 산다 생각하고 매주 만원씩 아무 코인이나 사면 뭐 하나는 터지지 않을까" 

평균 수익률을 따지자면 당연히 비트코인에 투자하는 것이 가장 높은 수익률을 보이겠지만,
우리가 원하는건 '큰 수익'이니 잭팟이 터질 조그만 확률이라도 있다면 로또를 사는 것 같은 마음가짐으로 시도해볼 수 있는 방법인 것 같습니다.

과연 이 '코인 로또'가 일정 수준의 수익을 낼 확률이 얼마인지 계산하고 싶었는데, 그러기 위해서는 여러 암호화폐의 과거 거래 데이터를 수집할 필요가 있었습니다.
이 포스트에서는 [Investing.com](https://kr.investing.com)에서 제공하는 과거 데이터를 Http request를 이용해 수집하는 과정을 담아보았습니다. 

### Rust reqwest

보통 크롤링은 Python을 많이 쓰지만, 요즘 Rust에 빠져있는 저는 여기서도 Rust를 이용했습니다.
Rust에서 HTTP request에 관한 기능을 제공하는 crate는 [*http*](https://docs.rs/http/0.2.3/http/)입니다. 
코인들의 과거 가격정보는 "https://kr.investing.com/crypto/{cryptoName}/historical-data"에서 확인할 수 있는데요.
하나하나 손으로 확인할 수 없으니 코인들의 리스트가 있는 "/crypto/currencies" 링크를 먼저 크롤링한 후 하나씩 반복문을 돌렸습니다. 

단계를 밟아가자면 먼저 사이트의 html을 모두 받습니다. 
여기서 Header를 수정하지 않으면 '404 Forbidden' 에러가 나기 때문에 헤더 수정해 요청합니다.

~~~ rust
use reqwest::header::USER_AGENT;

let client = reqwest::blocking::Client::new();
let mut res = client.get("https://kr.investing.com/crypto/currencies")
                .header(USER_AGENT, "Agent name")
                .send().unwrap();
~~~

그렇게 돌아온 반응을 문자열로 변환시켜줍니다. 

~~~ rust
let mut body = String::new();
res.read_to_string(&mut body).unwrap();
~~~

이렇게 변환된 문자열을 분석하기 편하게 정리해줘야하는데, rust에서는 [*select*](https://docs.rs/select/0.5.0/select/) crate가 있습니다.

~~~ rust
use select::document::Document;
use select::predicate::{Attr, Class, Name, Predicate};

let document = Document::from(body.as_str());
~~~

우리가 원하는 암호화폐 리스트는 "js-all-crypto-table" class에 소속되어 있는 tbody 부분입니다.
각 코인은 tr로 묶여 있습니다.

~~~ rust
let coin_table = document.find(Class("js-all-crypto-table")).next().unwrap()
                             .find(Name("tbody")).next().unwrap();
                             
for coin in coin_table.find(Name("tr")){
    // 코인 별 반복문 
}
~~~

이렇게 분류해낸 코인의 번호, 이름, 상대 경로, 절대 경로를 읽어서,

~~~ rust
let rank = coin.find(Class("rank")).next().unwrap().text();
let name = coin.find(Class("cryptoName")).next().unwrap().attr("title").unwrap();
let rel_link = coin.find(Name("a")).next().unwrap().attr("href").unwrap();
let link = format!("https://kr.investing.com/{}/historical-data", rel_link);
~~~




