---
title: "Implement generator in Rust using Genawaiter crate"
date: 2021-12-30-T14:15:30-14:30
categories:
  - Programmings
  - Statistical Physics
tags:
  - Simulation
  - Rust
  - Generator
  - Genawaiter
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

프로그래밍 언어에서 Iterator는 특정한 수열을 순차적으로 제공하는 type을 지칭합니다.
예를 들어 주어진 array(혹은 vector)에 속한 값들을 index 순으로 제공하는 아래의 코드들 역시 iterator를 이용한 코드입니다. 

```rust
// Rust code
let list = vec![0, 1, 2, 3];
for v in list{
    // v = 0, 1, 2, 3
}
```

```python
# Python code
list = [0, 1, 2, 3]
for v in list:
  # v = 0, 1, 2, 3
```

Python에서도 유사하지만, Rust에서는 [Iterator](https://doc.rust-lang.org/std/iter/trait.Iterator.html) trait을 정의함으로써 어떤 구조체던 Iterator로써 사용할 수 있습니다.
Iterator trait에는 여러 함수들이 존재하지만, 새로운 구조체에 Iterator trait을 정의할 때는 오직 iterator가 내뱉는 값의 type인 ``Item``과 전달할 다음 값을 반환하는 함수 ``next``를 정의하기만 하면 나머지는 자동적으로 완성되는 형식입니다. 
```rust
impl Iterator for MyStruct{
  type Item;
  
  fn next(&mut self) -> Option<Self::Item>
}
```

Iterator를 만드는 여러가지 방법이 있지만, 다른 언어에는 있지만 stable rust에는 없는 방법이 있습니다. 
바로 yield를 이용한 Generator입니다. 
Generator는 특정한 연산을 연속적으로 수행하면서 도출되는 값들을 순차적으로 반환해주는 iterator입니다.
기본적으로 제공되는 Iterator trait에는 '현재 상태'로부터 '다음 상태'를 유추할 수 있는 경우만 사용할 수 있지만,
Generator는 이전의 정보들을 모두 가지고 출력할 수 있기 때문에 보다 다양한 iterator를 만들 수 있습니다. 

아래의 python 코드는 0번째 피보나치 수부터 num-1번째 피보나치 수까지 출력하는 generator를 정의하는 코드입니다. 
```python
def getFibonnaciSeries(num):
    c1, c2 = 0, 1
    count = 0
    while count < num:
        yield c1
        c3 = c1 + c2
        c1 = c2
        c2 = c3
        count += 1
for i in getFibonnaciSeries(7):
    print(i)
``` 

물론 rust에서도 iterator에 ``c1, c2, count``를 구조체에 저장해둠으로써 같은 기능을 제공할 수는 있습니다.
```rust
struct FibIterator{
  c1 : usize,
  c2 : usize,
  count : usize,
}

impl FibIterator{
  fn new() -> Self{
    Self{ c1 : 0, c2 : 1, count : 0 }
  }
}

impl Iterator for FibIterator{
  type Item = usize;
  
  fn next(&mut self) -> Option<Self::Item>{
    if count == num{
      return None;
    } else {
      let temp = c1;
      c2 = c1 + c2;
      c1 = c2;
      count += 1;
      Some(temp);
    }
  }
}
```

하지만 generator의 구조가 복잡해질수록 iterator의 field가 많아진단 단점 때문에 모든 generator를 다른 구조의 iterator로 바꿀 수는 없습니다. 
그럼에도 rust에서는 generator를 stable 모드에서는 제공하지 않습니다.
그래서 generator를 이용하지 않고 여러가지 다른 [방법]("https://stackoverflow.com/questions/16421033/lazy-sequence-generation-in-rust")을 이용해 비슷한 방식을 구현하곤하죠. 
정확하게 generator와 yield가 왜 stable mode에서 쓰이지 못하는지는 따라가지 못했으나, 예상컨데 메모리 안정성에 문제가 생기기 때문이 아닐까 싶습니다. 
generator는 "resumable function"으로도 불리며, 함수가 돌아가다가 값이 반환되면 멈추고, 다시 함수가 "resume" 되면 앞서 멈춰선 계산 이후부터 다시 시작하는 방식을 가지게 됩니다.
즉, generator 내부 계산과 외부의 계산이 병렬적으로 이뤄지되 generator는 다시 호출되기 전까지는 멈춰서있는 구조로 이해할 수 있습니다. 
generator가 외부에서 정의된 변수를 사용하는 경우 해당 변수는 'shared mutable reference'가 되는 것이고 rust는 이러한 경우를 엄격하게 제한하고 있기 때문에 쓰이지 못한 것이 아닐까 생각합니다. 

그래서 지금의 rust 버젼에서는 nightly feature로만 generator가 존재하고, stable에서 이용하기 위해서는 genawaiter crate를 이용해야합니다. 
genawaiter crate는 [await / async](https://rust-lang.github.io/async-book/03_async_await/01_chapter.html)를 이용해 generator 기능을 구현한 crate입니다. 
이 문서에서는 spin chain에서 [hamiltonian matrix를 대각화하는 crate]("https://github.com/key262yek/exact_diagonalization")에서 genawaiter를 이용해 hamiltonian 구성을 간소화한 과정을 설명하고자 합니다. 

먼저 시스템에 존재하는 입자들의 spin state는 $z$방향으로 up spin / down spin으로 존재할 수 있는데, up spin을 1로 down spin을 0으로 나타내면 $N$개 입자가 존재하는 시스템의 상태는 $N$-bit 숫자로 나타낼 수 있습니다.
\begin{equation}
    \ket{26} = \ket{11010}
\end{equation}
또한 입자의 spin이 up spin과 down spin 사이를 오고가는 것은 bit flip을 통해 표현할 수 있습니다.

다음으로 Hamiltonian 구성을 알기 위해 spin-XXZ model의 Hamiltonian을 살펴봅시다. 
\begin{equation}
    H_0 = - \frac{J}{2} \sum_{n=0}^{L - 1}[\sigma_n^x \sigma^x_{n+1} + \sigma_n^y \sigma^y_{n+1} + \Delta_0 \sigma_n^z \sigma^z_{n+1}]
\end{equation}
여기서 $\sigma^x_n, \sigma^y_n, \sigma^z_n$는 각각 $n$번째 입자의 $x, y, z$방향 spin을 의미합니다.
$\sigma^x_n$와 $\sigma^y_n$는 raising, lowering operator $\sigma_n^{\pm} = (\sigma_n^x \pm i \sigma_n^y) / 2$를 이용해 아래와 같이 적힙니다. 
\begin{equation}
    H_0 = -J\sum_{n=0}^{L - 1} \qty[\sigma_n^{+} \sigma^{-}_{n+1} + \sigma_n^{-} \sigma^{+}_{n+1} + \frac{\Delta_0}{2} \sigma_n^z \sigma^z_{n+1}]
\end{equation}
이를 설명하자면 앞의 두 항은 $n$번째 입자와 $n+1$번째 입자의 $z$방향 spin이 서로 다른 경우에 한해서 $-J$의 에너지가 존재함을 의미하며, 시스템의 양자상태는 Hamiltonian에 의해 $n$번째 입자와 $n+1$번째 입자의 spin이 서로 교환된 상태가 됩니다. 
마지막 항은 두 인접한 입자의 $z$방향 spin이 같으면 $- J \Delta_0 / 2$가 추가되고, 서로 다른 경우 $J \Delta_0 / 2$가 추가되며, 이 경우엔 시스템의 양자상태가 변화하지 않습니다.
bit를 이용해 설명하자면 5개 입자 중 3개 입자가 up spin인 상태 $\ket{11010}$는 Hamiltonian이 적용됨에 따라 아래와 같이 변합니다. 
\begin{equation}
    H_0 \ket{26 = 11010} = - \frac{3}{2} \Delta_0 J\ket{26 = 11010} - J\ket{22 = 10110} -J \ket{28 = 11100} -J \ket{25 = 11001} -J \ket{11 = 01011}
\end{equation}
$H_0$의 행렬 표현에서 $i,j$ 요소는 $\bra{i}H_0\ket{j}$로 주어지며, 이를 구하기 위해서는 숫자 $j$가 주어졌을 때 $H_{0, ij}$가 $0$이 되지 않는 $i$를 출력해줄 수 있어야합니다. 
바로 이 지점이 generator를 이용하고자 하는 부분입니다. 
숫자 $j$가 들어왔을 때 $i$와 $H_{0, ij}$ 값을 계속 내뱉는 generator를 만들어봅시다.

```rust
// src/hamiltonian/mod.rs

use genawaiter::{sync::gen, yield_};

pub struct PeriodicNearestXXZ{
    pub delta_x : f64,
    pub delta_z : f64,
}

impl PeriodicNearestXXZ{
    pub fn new(delta_x : f64, delta_z : f64) -> Self{
        Self{
            delta_x,
            delta_z,
        }
    }

    pub fn apply_to<'a, S>(&'a self, state : &'a S) -> impl Iterator<Item = (usize, f64)> + 'a
        where S : State {
        gen!({
            let num = state.rep();
            let mut sum = 0f64;
            for ((i, si), (j, sj)) in state.periodic_pair_enumerate(){
                if si == sj{
                    sum -= self.delta_z / 2f64;
                } else {
                    sum += self.delta_z / 2f64;
                    let flipped = bit_flip_unsafe(num, i, j);
                    yield_!((flipped, -self.delta_x));
                }
            }

            yield_!((num, sum));
        }).into_iter()
    }
}
```

하나하나 뜯어봅시다.
```rust
use genawaiter::{sync::gen, yield_};
```
우리는 genawaiter crate에서 `gen` macro와 `yield_` macro를 사용할겁니다.

```rust
pub struct PeriodicNearestXXZ{
    pub delta_x : f64,
    pub delta_z : f64,
}
```
먼저 Hamiltonian의 특성을 저장할 struct을 새로 정의해줍니다. 우리는 XXZ model을 사용할테니 엄밀히는 delta_z만 필요로하지만, 혹시 몰라 delta_x와 delta_z를 모두 갖도록 했습니다.

```rust
pub fn new(delta_x : f64, delta_z : f64) -> Self{
    Self{
        delta_x,
        delta_z,
    }
}
```
struct를 생성하는 함수이고,

```rust
pub fn apply_to<'a, S>(&'a self, state : &'a S) -> impl Iterator<Item = (usize, f64)> + 'a
    where S : State{
    // Something
}
```
이것이 비로소 Hamiltonian을 특정한 state에 적용하는 함수입니다.
crate에서는 State trait이 존재하고, 여기에 해당하는 어떤 struct도 대상이 될 수 있도록 하기 위해서 trait bound에 State를 추가해줍니다.
더 중요한 것은 lifetime입니다. 
우리는 self와 state의 reference만을 받아 연산할 것이고, generator의 특성상 iterator가 곧바로 소진되는 것이 아니다보니 출력되는 iterator의 lifetime보다 self와 state의 reference가 lifetime이 길 필요가 있습니다. 
이를 위해 lifetime을 명시해줍니다. 

```rust
gen!({
    // Something
}).into_iter()
```
gen macro는 code block을 받아 generator로 만들어주는 macro이고,
말미에 붙은 into_iter() 함수를 통해 우리가 코드에서 사용하기 편한 iterator의 형태로 변환시켜주어 반환할 예정입니다.


```rust
let num = state.rep();
let mut sum = 0f64;
for ((i, si), (j, sj)) in state.periodic_pair_enumerate(){
    if si == sj{
        sum -= self.delta_z / 2f64;
    } else {
        sum += self.delta_z / 2f64;
        let flipped = bit_flip_unsafe(num, i, j);
        yield_!((flipped, -self.delta_x));
    }
}
yield_!((num, sum));
```
Code block의 내부에서는 state의 representation number를 num에 받고,
bit 숫자를 iterate합니다. 
이 기능을 담당하는 iterator를 호출해주는 것이 periodic_pair_enumerate()입니다. 
$i$번째 bit $s_i$와 $j$번째 bit $s_j$를 출력해주는데, 어차피 인접한 bit임에도 $i,j$로 나눈 이유는 시스템이 periodic하기 때문에 $(L, 0)$ pair도 존재하기 때문입니다.
$s_i$와 $s_j$를 비교했을 때 다른 경우엔 bit를 바꾸어 숫자와 에너지 값을 yield해줍니다.
그리고 diagonal 항에 해당하는 값은 계속 sum에 더해준 후 맨 마지막에 state 자기 자신의 representation 숫자와 sum을 반환하고 iterator는 끝나게 됩니다.

이런 식으로 코드를 짜게 되면 실제 계산 코드에서는 아래와 같이 사용할 수 있습니다.
```rust

let xxz = PeriodicNearestXXZ::new(1f64, delta);
let mut hamiltonian : Array2<Complex64> = Array2::zeros((n, n));

for (rep, value) in xxz.apply_to(state){
    hamiltonian[[rep, state.rep()]] += value;
}

```

물론 generator 내부 code block이 그다지 복잡하지 않으니 굳이 rust에서 구현하기 어려운 generator까지 이용해 함수를 선언해야 하느냐 물으신다면 그럴 필요가 없을 수도 있겠습니다만, 공부를 겸하기도 하고, iterator의 맥락에서 훨씬 간단하게 코드를 읽을 수 있도록 하기 때문에 장단이 있지 않았나 생각합니다. 


