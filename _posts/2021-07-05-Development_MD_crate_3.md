---
title: "Crate development - System module II"
date: 2021-07-05-T21:35:30-22:00
categories:
  - Programmings
  - Statistical Physics
tags:
  - Simulation
  - Molecular Dynamics
  - Brownian Dynamics
  - Crate Development
  - Generic Type
  - Procedural Macros
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---


## Product topology
지난 [post]({% post_url 2021-06-25-Development_MD_crate_2 %})에서 System module의 기본적인 구조를 잡고, Continuous system, Lattice system, Network system에서의 topology를 정의해주었습니다. 다음 단계로 확장성을 위해 product topology에 대한 topology trait을 구현해놓고자 했습니다.

가장 간단한 방식으로 generic type을 이용해 tuple의 trait을 정의하는 방식입니다. 
```rust
impl<P1 : Point, P2 : Point> Point for (P1, P2){}

impl<P1, T1, P2, T2> Topology<(P1, P2)> for (T1, T2)
    where P1 : Point,
          T1 : Topology<P1>,
          P2 : Point,
          T2 : Topology<P2>{

    fn check_move(&self, pos : &(P1, P2), movement : &(P1, P2)) -> bool{
        return self.0.check_move(&pos.0, &movement.0) || self.1.check_move(&pos.1, &movement.1);
    }
}
```

여기에는 한 가지 문제가 있는데, 사용성의 측면에서 tuple struct를 새로 정의하는 경우를 포괄하지 못한단 점입니다. 
tuple과 tuple struct는 다르기 때문이죠. 
```rust
struct Lane(Cartessian1D<i32>, Cartessian1D<f64>);
struct LaneTopology(LatticeTopology, ContinuousTopology);

// (define topology variable and positions);
sys.check_move(&point, &movement);   // Cannot compile because check_move is not developed
```

결국 새로운 tuple struct를 정의할 때마다 새롭게 trait implementation code를 적어줘야한단 의미인데, 기본적으로 tuple struct는 tuple과 동일하기 때문에 간단한 코드를 굳이 반복적으로 적어야한다는 불편함이 생깁니다. 그러니 이를 macro를 통해 간소화하는 편이 더 나아보입니다. 

## Derive macros

이럴 때 사용할 수 있는 것이 derive macro입니다. 
rust 코드를 사용하고 계시다면, 새로운 struct를 정의할 때 아래와 같은 코드를 작성해보셨을겁니다.
```rust
#[derive(Copy, Clone, Debug, PartialOrd, Hash, PartialEq)]
struct NewStruct{
  ...
}
```
여기서 `#[derive(...)]` 부분이 바로 derive macro입니다.
해당 struct에 copy, clone 등의 trait을 자동으로 구현해주는 macro죠. 
이는 procedural macros의 일환으로 이에 관해서는 이미 [다른 crate]({% post_url 2021-05-07-Procedure_Macros %})를 구현하면서 사용한 적 있습니다.
이번에도 이를 이용해 fancy한 trait implementation을 완성해봅시다. 

```rust
extern crate proc_macro;
use proc_macro::TokenStream;
use proc_quote::{quote, quote_spanned};
use syn::spanned::Spanned;
use syn::{parse_macro_input, DeriveInput, Data, Fields, Index};

/// Derive macros autocomplete the trait Point.
///
/// MoldyBrody에서는 Point trait이 존재하는데, 이를 자동으로 구성해주는 매크로이다.
#[proc_macro_derive(Point)]
pub fn derive_point(item: TokenStream) -> TokenStream {
    let x = syn::parse_macro_input!(item as DeriveInput);
    let name = x.ident;

    let tokens = proc_quote::quote!{
        impl Point for #name {}
    };
    tokens.into()
}


/// Derive macros autocomplete the trait Topology.
///
/// Topology trait을 autocomplete 해주는 매크로이다.
/// Topology trait은 대응하는 Point generic type이 특정되어야하는데,
/// 이를 위해서 PointName이라는 attributes를 받는다.
#[proc_macro_derive(Topology, attributes(PointName))]
pub fn derive_topology(item: TokenStream) -> TokenStream {
    let x = parse_macro_input!(item as DeriveInput);

    let name = x.ident;
    let point_name : syn::Ident = x.attrs[0].parse_args().unwrap();

    let tokens = match x.data{
        Data::Struct(ref data) => {
            match data.fields{
                Fields::Unnamed(ref fields) =>{
                    let recurse = fields.unnamed.iter().enumerate().map(|(i, f)| {
                        let index = Index::from(i);
                        quote_spanned! {f.span()=>
                            self.#index.check_move(&pos.#index, &movement.#index)
                        }
                    });
                    quote! {
                        impl Topology<#point_name> for #name{
                            fn check_move(&self, pos : &#point_name, movement : &#point_name) -> bool{
                                false #(|| #recurse)*
                            }
                        }
                    }
                },
                _ => unimplemented!(),
            }
        },
        Data::Enum(_) |
        Data::Union(_) => unimplemented!(),
    };
    // panic!("{}", tokens.to_string());
    tokens.into()
}
```

위와 같이 procedural macro를 만들어서 아래와 같은 코드를 쓸 수 있게 되었습니다.
```rust
#[derive(Point)]
pub struct Lane(Cartessian1D<i32>, Cartessian1D<f64>);

#[derive(Topology)]
#[PointName(Lane)]
pub struct LaneTopology(LatticeTopology, ContinuousTopology);
```
한가지 아쉬운 점은 derive macro는 그 안에서 다시 attribute를 주지 못해서 PointName attribute를 따로 주어야하고, 이 때문에 완벽하게 좋은 사용성을 기대하긴 어려워졌습니다. 




