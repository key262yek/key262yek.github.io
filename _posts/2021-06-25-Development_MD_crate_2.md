---
title: "Crate development - System module"
date: 2021-06-25T21:35:30-22:00
categories:
  - Programmings
  - Statistical Physics
tags:
  - Simulation
  - Molecular Dynamics
  - Brownian Dynamics
  - Crate Development
  - Generic Type
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

## Structure of the system module

첫번째 System module을 정의하려고 하는 중입니다. 
처음 만드려는 구조는 아래와 같이 Generic type로 서로를 호출하는 방식이었습니다.
```rust
trait Point<N : Neighbor<P>>{}
trait Neighbor<P : Point<N>>{}
```
하지만 이 방식은 아예 rust 문법에 맞지 않아서 method 단계로 generic type을 내려서 코딩을 시도했습니다. 
```rust
trait Point{
  fn neighbor<N : Neighbor>(&self) -> N;
}

trait Neighbor{
  fn inclusion<P : Point>(&self, pos : &P) -> bool;
}
```
하지만 이렇게 하더라도 제가 원하는 구조를 맞추기는 불가능했습니다. 
제가 원하는 구조는 Point trait을 가지는 특정한 structure P와 Neighbor trait을 가지는 특정한 structure N 사이 짝을 이루는 것인데,
위와 같이 짜면 P는 generic neighbor N에 대해 함수를 써야하고,
N은 generic point P에 대해 함수를 써야해서 특정한 두 구조체 사이 관계식을 정의하지 못하게 됩니다.
즉, Point trait을 가지는 다른 구조체 P2가 있을 때
Neighbor의 inclusion은 P 뿐 아니라 P2도 포괄할 수 있는 함수를 작성해야만 합니다. 
그러면 구조체들이 고유하게 가지고 있는 기능들은 전혀 사용할 수 없게 됩니다. 

해결책은 의외로 간단했습니다. 
실제 Topology의 구조를 그대로 따라하면 됩니다. 
Topology는 본래 점들의 집합과 Basis 집합의 pair로 정의되는데요. 
Point와 Neighbor가 쌍을 만들어야 비로소 Topology로 정의될 수 있습니다. 
아직 마땅한 기능을 필요로 하지 않는 Point와 Neighbor는 빈 Trait으로 정의하고, Topology를 아래와 같이 정의할 수 있습니다. 

``` rust
/// Point in the system
///
/// Topology를 구성하는 요소 중 점에 해당하는 trait.
/// 각 점에 대응되는 이웃을 알 수 있어야합니다.
pub trait Point{
}

/// Neighbor of a point
///
/// Topology를 구성하는 요소 중 이웃에 해당하는 trait.
/// 특정 점이 이웃에 속하는지 여부를 확인할 수 있어야 합니다.
pub trait Neighbor<'a>{
}


/// Topology on the system
///
/// Point와 Neighbor trait을 구성요소로 하여 시스템의 Topology를 정의해주는 trait입니다.
/// 점과 이웃의 관계는 비로소 Topology 안에서 정의됩니다.
pub trait Topology<'a, P, N>
    where P : Point,
          N : Neighbor<'a>{
    /// Return a neighbor of the point
    ///
    /// 점의 이웃을 알려주는 함수입니다.
    /// 시뮬레이션에서 이웃의 보다 실천적 의미는 한 번에 갈 수 있는 점들의 집합을 의미합니다.
    fn neighbor<'b : 'a>(&self, pos : &'b P) -> N;

    /// Check whether a point is in the neighbor or not
    ///
    /// point가 neighbor에 속하는지 여부를 확인하는 함수입니다.
    /// particle이 특정 위치로 이동할 수 있는지 여부를 test할 때 쓰입니다.
    fn inclusion(&self, pos : &P, neigh : &N) -> bool;
}
```

여기서 lifetime 문제가 조금 중요해집니다. 
실공간에서 보통 neighbor는 특정한 점을 중심으로 하는 open ball의 형태로 정의될 수 있는데,
이때 점의 좌표를 그대로 clone하는 것은 메모리 측면에서도, 속도 측면에서도 전혀 효율적인 방법이 아닙니다.
그래서 neighbor에는 좌표의 reference를 저장하는 방법울 택했고, implicit lifetime을 허용하지 않는 rust의 성격 때문에 Neighbor에도 lifetime identifier가 붙었습니다.

## Continuous Topology

먼저 정리한 것은 연속적인 공간에서의 open-ball topology였습니다.
각 점을 정의하고, 그 점을 중심으로 하는 특정 거리까지의 open ball을 neighbor로 정의하는 방식의 topology입니다. 
여기서 `특정 거리`는 시스템에서 입자가 움직일 수 있는 maximum step size를 나타냅니다.
우리는 연속적으로 움직이는 입자의 운동을 미소변화의 중첩으로 근사할 것이기 때문에, 오차를 줄이기 위해서는 너무 큰 step이 생기면 안됩니다.
따라서 특정한 limit을 system에 미리 부여해 이를 넘는 step이 벌어지지 않도록 코드 범위 안에서 확인할 필요가 있습니다. 

그런 관점에서 작성된 코드 구조는 아래와 같습니다. 
```rust
/// Cartessian coordinate of n-dimensional continuous system
#[derive(Clone, Debug, PartialEq, PartialOrd, Hash)]
pub struct CartessianND<T>{
    /// Coordinate of Point
    pub coord : Vec<T>,
}

impl Point for CartessianND<f64>{}

impl CartessianND<f64>{
    /// return a dimension of system
    pub fn dim(&self) -> usize{
        self.coord.len()
    }

    /// return a distance to other point
    pub fn distance(&self, other : &Self) -> f64{
        let dim = self.dim();
        if dim != other.dim(){
            panic!("{}", ErrorCode::InvalidDimension);
        }

        let mut r = 0f64;
        for i in 0..dim{
            let dx = self.coord[i] - other.coord[i];
            r += dx * dx;
        }

        return r.sqrt();
    }
}

/// Open-Ball neighborhood corresponds to [`Cartessian1D`](struct@Cartessian1D)
#[derive(Copy, Clone, Debug, PartialEq, PartialOrd)]
pub struct OpenBallND<'a>{
    /// Point of ball center
    pub center : &'a CartessianND<f64>,
}

impl Neighbor<'_> for OpenBallND<'_>{}

#[derive(Copy, Clone, Debug, PartialEq, PartialOrd)]
pub struct ContinuousTopology{
    /// Maximum step size available in the system
    max_step : f64,
}

impl<'a> Topology<'a, CartessianND<f64>, OpenBallND<'a>> for ContinuousTopology{
    /// Return a openball of center pos.
    fn neighbor<'b : 'a>(&self, pos : &'b CartessianND<f64>) -> OpenBallND<'b>{
        OpenBallND{
            center : &pos,
        }
    }

    /// Check whether the point is in a neighbor or not.
    fn inclusion(&self, pos : &CartessianND<f64>, neigh : &OpenBallND) -> bool{
        let r = pos.distance(neigh.center);
        r < self.max_step
    }
}
```

여기서 한 단계 더 변화한 것은 dimension에 관한 부분입니다.
위 처럼 위치를 Vector로 표시하면, 두 벡터를 같이 연산할 일이 생길 때 매번 dimension을 체크해야하는 불편함이 생깁니다.
if문이 하나 추가되는 일이기 때문에 그만큼 속도도 더 떨어집니다.
따라서 우리가 자주 쓰는 1D부터 4D 정도까지의 시스템은 vector가 아닌 fixed sized array로 정의해두는 편이 좋겠다 싶었습니다.
그렇게만 해두면 서로 다른 차원의 벡터가 서로 연산될 일은 compile 단계에서 걸러지므로 if문을 굳이 사용할 필요가 없습니다. 
다만 반복적인 코드를 여러번 작성하는 것도 일이므로 아래와 같은 macro를 통해 여러 좌표계를 정의해주었습니다. 

```rust
macro_rules! impl_cartessian_nD{
    ($type_name:ident, $dim:expr) =>{
        #[derive(Copy, Clone, Debug, PartialEq, PartialOrd, Eq, Ord, Hash)]
        pub struct $type_name<T>{
            /// Coordinate of Point
            pub coord : [T; $dim],
        }

        impl Point for $type_name<f64>{}

        impl $type_name<f64>{
            pub fn dim(&self) -> usize{
                $dim
            }

            pub fn distance(&self, other : &Self) -> f64{
                let mut r = 0f64;
                for i in 0..$dim{
                    let dx = self.coord[i] - other.coord[i];
                    r += dx * dx;
                }

                return r.sqrt();
            }
        }
    }
}

impl_cartessian_nD!(Cartessian2D, 2);

macro_rules! impl_neighbor_cartessian_nD{
    ($neighbor_name:ident, $cartessian_name:ident, $dim : expr) =>{
        #[derive(Copy, Clone, Debug, PartialEq, PartialOrd)]
        pub struct $neighbor_name<'a>{
            /// Point of ball center
            pub center : &'a $cartessian_name<f64>,
        }

        impl Neighbor<'_> for $neighbor_name<'_>{}

        impl<'a> $neighbor_name<'a>{
            pub fn new(center : &'a $cartessian_name<f64>) -> Self{
                $neighbor_name{
                    center,
                }
            }
        }
    }
}

impl_neighbor_cartessian_nD!(OpenBall2D, Cartessian2D, 2);

macro_rules! impl_continuous_topology_nD{
    ($cartessian_name:ident, $neighbor_name:ident, $dim:expr) =>{
        impl<'a> Topology<'a, $cartessian_name<f64>, $neighbor_name<'a>> for ContinuousTopology{
            fn neighbor<'b : 'a>(&self, pos : &'b $cartessian_name<f64>) -> $neighbor_name<'b>{
                $neighbor_name{
                    center : &pos,
                }
            }
                
            fn inclusion(&self, pos : &$cartessian_name<f64>, neigh : &$neighbor_name) -> bool{
                let r = pos.distance(neigh.center);
                r < self.max_step
            }
        }
    }
}

impl_continuous_topology_nD!(Cartessian2D, OpenBall2D, 2);
```

이 과정에서 doc test를 위한 comment가 들어갈 필요가 있었는데,
macro는 comment 내부까지 값을 parsing해주지는 않습니다. 
따라서 [doc_comment](https://docs.rs/doc-comment/0.3.3/doc_comment/)라는 crate가 제공하는 macro를 추가적으로 사용해 document까지 만들 수 있었습니다.
doc_comment macro가 어떻게 작동하는지는 [repository](https://github.com/key262yek/moldybrody/blob/master/src/system/point.rs)를 직접 확인해주세요.

## Remove neighbor trait

그런데 neighbor를 이렇게 정의해봐야 무슨 의미가 있나 싶었어요.
그냥 reference를 저장하는 변수가 굳이 의미를 가지기 어려워보였거든요. 
neighbor의 가장 중요한 역할은 특정한 movement가 가능한 move인지 아닌지를 판가름하는 기준이 되는 것입니다. 
하지만 continuous system이나 lattice에서는 두 점 사이 distance로 test하면 되고,
network에서는 system에 adjoint matrix를 정의해두어야하기 때문에 이를 기준으로 test하면 될거라 생각이 됩니다.
그러면 여기서 neighbor가 들어올 부분이 전혀 없는거죠. 
그래서 neighbor trait을 제거하고 다시 코드를 짰습니다. 

```rust
/// Point in the system
pub trait Point{}

/// Cartessian coordinate of 1-dimensional continuous system
#[derive(Copy, Clone, Debug, PartialEq, PartialOrd, Eq, Ord, Hash)]
pub struct Cartessian1D<T>{
    /// Coordinate of Point
    pub coord : T,
}

impl Point for Cartessian1D<f64>{}

impl Cartessian1D<f64>{
    /// return a dimension of system
    pub fn dim(&self) -> usize{
        1
    }

    /// return a distance to other point
    pub fn distance(&self, other : &Self) -> f64{
        (self.coord - other.coord).abs()
    }

    /// return a norm of vector
    pub fn norm(&self) -> f64{
        return self.coord.abs();
    }
}

/// Topology on the system
pub trait Topology<P>
    where P : Point{
    /// Check whether a movement is valid or not.
    fn check_move(&self, pos : &P, movement : &P) -> bool;
}

/// Normal openball topology for continuous system
#[derive(Copy, Clone, Debug, PartialEq, PartialOrd)]
pub struct ContinuousTopology{
    /// Maximum step size available in the system
    max_step : f64,
}

impl Topology<Cartessian1D<f64>> for ContinuousTopology{
    /// Check whether the movement is valid or not
    fn check_move(&self, _pos : &Cartessian1D<f64>, movement : &Cartessian1D<f64>) -> bool{
        let r = movement.norm();
        r < self.max_step
    }
}
```

## Lattice Topology

Lattice는 사실상 거의 다 개발되어 있는 상태입니다.
`Cartessian<i32>`을 `Cartessian<f64>`를 개발할 때 같이 개발했거든요.
여기서 taxi_distance나 taxi_norm을 정의해두었고, 이를 토대로 topology를 다 정의할 수 있었습니다.

```rust
impl Topology<CartessianND<i32>> for LatticeTopology{
    /// Check whether the movement is valid or not
    fn check_move(&self, pos : &CartessianND<i32>, movement : &CartessianND<i32>) -> bool{
        if pos.dim() != movement.dim(){
            panic!("{}", ErrorCode::InvalidDimension);
        }
        let r = movement.taxi_norm();
        r <= self.max_step
    }
}
```

## Network Topology

Network의 topology는 point부터 개발해야합니다. 
여기서 Point는 index만 주어지면 될 것이고, 연결 정보는 topology structure에 담으면 되리라 생각합니다. 
이때 network의 edge에 weight가 있을 수 있으므로 여러 변수가 가능한 generic type으로 network를 정의했고, 0 혹은 false가 끊어져있는 값이 될테니 default를 이용했습니다. 

```rust
/// node of a network
#[derive(Copy, Clone, Debug, PartialEq, PartialOrd, Eq, Ord, Hash)]
pub struct NodeIndex{
    /// Index of node
    pub index : usize,
}

impl Point for NodeIndex{}

/// Network topology
#[derive(Clone, Debug, PartialEq, PartialOrd)]
pub struct NetworkTopology<T>{
    /// The number of nodes
    num_node : usize,
    /// Adjoint matrix,
    adjoint : Vec<Vec<T>>,
}

impl<T : Copy + Default + PartialEq> Topology<NodeIndex> for NetworkTopology<T>{
    /// Check whether the movement is valid or not
    fn check_move(&self, depart : &NodeIndex, arrival : &NodeIndex) -> bool{
        let connect = self.adjoint[depart.index][arrival.index];
        !(connect == T::default())
    }
}
```







