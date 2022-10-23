---
title: "Performance gap between Array and Vector in Rust"
date: 2022-01-30-T14:15:30-14:30
categories:
  - Programming
  - Statistical Physics
tags:
  - Simulation
  - Rust
  - Allocation
  - Heap
  - Stack
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

[지난 포스트]({% post_url 2021-06-05-Development_MD_crate_1 %})에서 설명한 바와 같이 Molecular dynamics, Brownian dynamics 시뮬레이션을 위한 패키지 [moldybrody]("https://github.com/key262yek/moldybrody")를 작성하고 있습니다. 
패키지에서 가장 근간을 이루는 것은 벡터 구조체와 그의 연산인데, 크게 Array를 이용하는 `Cartessian<T, N>`과 Vector를 이용하는 `CartessianND<T>`가 있습니다. 
여기서 `T`는 벡터의 성분의 자료형을 나타내는 Generic이고, `N`은 compile time에 정의되는 상수로 Array의 길이를 지칭합니다. 
이 둘을 처음에 나눈 이유는 compile time에서 벡터의 차원이 명확하게 정의된다면 서로 다른 차원의 벡터가 연산되는 식의 에러를 미리 없앨 수 있기 때문이었습니다. 
하지만 거의 개발이 마무리되가는 시점에 벡터연산의 퍼포먼스를 확인하니 Array는 성능을 위해서라도 포기할 수 없는 자료형식인 것 같습니다.
오늘은 이렇게 판단하게 된 과정에 대해 기록을 남겨두려고 합니다.

### Initial code structure
처음의 코드는 아래와 같은 형식을 띄고 있었습니다.

```rust
use std::fmt::Debug;
use std::convert::TryInto;

impl<T : Scalar, const N : usize> Cartessian<T, N>{
    pub fn map<'a, B, F>(&'a self, mut f : F) -> Cartessian<B, N>
        where F : FnMut(&'a T) -> B,
              T : Clone,
              B : Debug{
        
        Cartessian::<B, N>{
            coord : self.into_iter().map(|x| f(x)).collect::<Vec<B>>().try_into().unwrap(),
        }
    }
}

impl<T : Scalar, const N : usize> Add<Cartessian<T, N>> for &Cartessian<T, N>{
    type Output = Cartessian<T, N>;
    
    fn add(self, rhs : Cartessian<T, N>) -> Cartessian<T, N>{
        let mut out = self.clone();
        out.iter_mut().zip(&rhs).for_each(|(x, y)| *x = *x + *y);
        out
    }
}

impl<'a, T : Scalar, const N : usize> Mul<T> for &'a Cartessian<T, N>{
    type Output = Cartessian<T, N>;
    
    fn mul(self, c : T) -> Cartessian<T, N>{
        self.map(move |elt| *elt * c);
    }
}

fn compute_move(){
    // ... 
    let dv = force * (dt / self.mass());
    let dx = self.vel() * dt + force * (dt * dt / (self.mass() + self.mass()));
    return (dx, dv);  
}
```

부연설명을 하자면 `Cartessian<T, N>`의 덧셈과 상수곱을 사용했고, 이들에 관한 Trait이 정의되어 있는데 여기서 상수곱에서는 각 성분에 상수를 곱하는 과정을 map 함수를 통해 구현했고, 이후에 collect 함수를 통해 iterator에서 Vec를 만들고, Vec를 try_into 함수를 통해 array로 변환하는 방식으로 상수곱의 결과물을 출력했습니다.
    
당연히 Performance가 매우 떨어졌고, 가장 오래걸리는 지점은 Allocation이었습니다. 횟수로는 대략 1200만번의 Allocation이 있었습니다. 
그래서 이 부분의 코드를 수정했습니다.

### First Improvement

그렇게 수정된 코드가 아래의 코드입니다. 

```rust

impl<'a, T : Scalar, const N : usize> AddAssign<&'a Cartessian<T, N>> for Cartessian<T, N>{
    
    fn add_assign(self, rhs : Cartessian<T, N>) -> Cartessian<T, N>{
        for (x, y) in self.iter_mut().zip(rhs){
            x.add_assign(*y);
        }
    }
}

impl<'a, T : Scalar, const N : usize> MulAssign<T> for Cartessian<T, N>{
    
    fn mul_assign(self, c : T) -> Cartessian<T, N>{
        for (x, y) in self.iter_mut().zip(rhs){
            x.mul_assign(*y);
        }
    }
}

fn compute_move(){
    // ..
    
    let mut temp = Cartessian::<T, N>::default();
    temp += force;
    temp *= dt / self.mass();
    let dv = temp.clone();
    
    let mut dx = Cartessian::<T, N>::default();
    dx += self.vel();
    dx *= dt;
    
    temp *= T::zero();
    temp += force;
    temp *= dt * dt / (self.mass() + self.mass());
    dx += temp;
    
    return (dx, dv);  
}
```

매우 조악하지만 AddAssign과 MulAssign trait을 통해 allocation을 줄이려고 했습니다. 
실제로 코드가 돌아가는 과정에서 Heap allocation이 거의 일어나지 않는 것을 확인할 수 있었습니다.
총 allocation은 26번 정도 있었습니다. 속도도 3배 가량 빨라졌습니다.

### Realize a Problem

하지만 이 결과는 이상했습니다.
실제로 저는 clone 함수를 이용하고 있으니 array의 allocation은 분명 존재해야 합니다.
하지만 Allocation에는 count가 되지 않고, 심지어 time profiler에서도 heaviest stack에서 allocation은 찾아볼 수 없었습니다. 
Clone에서 Allocation을 하지 않는 경우도 있나? 싶었지만 관련한 정보가 나오지 않았죠.

차이점은 `Heap`에 있었습니다.
지금까지 Stack memory와 Heap memory를 너무 피상적으로만 이해하고 있다보니, 당연히 Heap allocation에서는 Stack allocation이 가산되지 않는단걸 떠올리지 못했던거죠.
쨌든, Array를 사용하는 `Cartessian<T, N>`은 compile time에서 그 크기가 결정되고, 이후에 변경될 일도 없기 때문에 Heap이 아닌 Stack에 allocate되게 됩니다. 
그렇기 때문에 clone이 매우 빠르게 일어날 수 있는 것이죠.

### Improve Performance on Trait Implementation

그렇다면 왜 처음의 코드는 그렇게나 오래걸렸던 것일까.
문제는 iterator가 vector를 들렸다가 array로 돌아오는 구조 때문에 그렇습니다. 
이렇게 둘러오게 되면 Heap allocation이 두 번 존재하게 되니 불필요한 시간 소모가 매우 커지는 것이죠.
그렇기에 코드 전반에 걸쳐 map 함수의 사용을 모두 제거하고, clone을 통해 이용하는 방식을 채택하게 됩니다.
또 종종 이미 allocate된 변수가 borrow가 아니라 완전히 ownership을 넘기는 연산의 경우 (아래 code의 벡터 덧셈) 기존에 allocate된 변수에 값만 바꿔 반환하는 형태로 stack allocation도 줄이는 코드를 작성했습니다.

```rust
impl<T : Scalar, const N : usize> Add<Cartessian<T, N>> for Cartessian<T, N>{
    type Output = Cartessian<T, N>;
    
    fn add(mut self, rhs : Cartessian<T, N>) -> Cartessian<T, N>{
        self.iter_mut().zip(&rhs).for_each(|(x, y)| *x += *y);
        self
    }
}

impl<'a, T : Scalar, const N : usize> Mul<T> for &'a Cartessian<T, N>{
    type Output = Cartessian<T, N>;
    
    fn mul(self, c : T) -> Cartessian<T, N>{
        let mut out = self.clone();
        out.iter_mut().for_each(|x| *x = *x * c)
        out
    }
}

fn compute_move(){
    // ... 
    let dv = force * (dt / self.mass());
    let dx = self.vel() * dt + force * (dt * dt / (self.mass() + self.mass()));
    return (dx, dv);  
}
```

이렇게 바꾸고 나니 불필요한 Heap allocation도 사라지고, 코드의 가독성도 올라갔습니다.
물론 동일한 코드를 `CartessianND<T>`에 대해서 구현하는 경우에는 기존과 다르지 않게 Heap allocation에 큰 시간을 소모하는 것을 확인할 수 있습니다.
당연히 이 경우는 크기가 가변적인 `Vec<T>` 구조의 clone이므로 Heap allocation이 일어나는것이죠.

### Conclusion

사실 저는 박사과정이 끝났을 때 프로그래머로 일하는 진로에 대해 고민하고 있습니다.
고민만 하고 공부는 소홀히 하니 이런 당연한 문제도 곧바로 이해하지 못하고 떠돌게 되는 것 같은데, 제대로 된 프로그래머로 일하려고 한다면 컴퓨터의 기본 구조, 특히 메모리 부분에 대해서 더 깊은 이해가 필요할 것 같습니다. 

