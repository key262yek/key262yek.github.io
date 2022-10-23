---
title: "Crate development - Planning"
date: 2021-06-05T21:35:30-22:00
categories:
  - Programming
  - Statistical Physics
tags:
  - Simulation
  - Molecular Dynamics
  - Brownian Dynamics
  - Crate Development
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

[moledyn](https://github.com/key262yek/moledyn) crate에 procedural macro까지 만들고나니 이보다 더 나은 구조로 다시 짜고 싶다는 생각을 떨치기가 어렵습니다.
나 외에 다른 사람이 사용하기 힘든 구조라는 것은 차치하더라도, 기존의 개발 방향성과도 많이 달라지지 않았나 생각해 다시 한 번 바닥부터 도전을 해보고자 합니다.
그러기 위한 계획을 먼저 세워봅시다.

## 만들고자 하는 기능

기본적으로 Molecular Dynamics(MD)와 Brownian Dynamics(BD)를 수행할 수 있는 플랫폼을 제공하는 것을 목적으로 합니다.
시뮬레이션 셋업의 모든 경우의 수를 다 구현할 수는 없을 것입니다.
시스템만 놓고 보더라도 좌표가 연속적인지, 이산적인지, 이산적인 경우에도 격자구조인지 complex network로 주어지는지, 시스템의 차원은 몇 차원인지 등이 달라질 수 있으며,
구현된 두 개의 시스템이 결합되어 하나의 시스템이 될 수도 있습니다.
(예를 들어, 1D line과 1D lattice를 결합하면 여러 차선이 있는 도로가 되는 것)
그렇기 때문에 큰 틀에서 crate의 내용은 아래와 같을 것입니다.

- 기초적인 기능을 선별해 Trait로 선언
- 자주 사용되는 객체들을 구현
- 새롭게 구현하는 것을 자동화하기 위한 procedural macro 구현

그럼 보다 자세하게 어떤 객체들이 구현되어야하는지 정리하기 위해 시뮬레이션이 어떤 구조로 진행되는지 짚어봅시다.

1. 입자들의 위치를 초기화합니다.
2. 입자들 사이 상호작용(BD의 경우 random force 포함)을 계산합니다.
3. 2의 결과와 입자들의 현재 상태를 이용해 입자들의 위치를 조금씩 변화시킵니다.
4. 입자들이 시스템의 경계조건을 위배하지는 않았는지 확인합니다.
5. 1-4를 반복하며 입자들을 계속 이동시키고, 측정하고자 하는 물리량을 출력합니다.

이 단계들을 위해서 필요한 요소들은 아래와 같을 것입니다.

- System : 입자들이 존재할 공간, 위치가 가지는 구조 내지는 topology를 결정한다. topology는 입자들의 위치 변화를 결정하는 중요한 요소이다. 
- Boundary condition : 입자들의 위치가 존재할 수 있는 범주를 결정하고, 경계를 맞닥뜨렸을 때 어떻게 처리할지를 결정한다. 
- Force computation : Global potential의 경우엔 입자의 위치, 입자간 상호작용의 경우엔 입자간 거리 등을 기준으로 입자가 받게 되는 힘의 크기를 결정한다. 
- Movement : 입자가 받는 힘, 입자의 현재 상태로부터 미소 시간동안의 입자의 위치변화를 근사합니다. 근사의 방법론은 어느 차수까지 사용하느냐에 따라 달라질 수 있습니다.
- Data import / export : 시뮬레이션을 위해 필요한 설정 정보들을 자동으로 알려주고, 이를 받아 시뮬레이션이 구동될 수 있도록 합니다. 그리고 시뮬레이션 결과 파일에 시뮬레이션 설정 정보와 측정 결과를 출력하고, 이후 분석시에 다시 읽어올 수 있어야합니다. 


### System
Continuous system, Lattice, Complex network, 그리고 lane과 같이 서로 다른 시스템을 결합한 시스템을 모두 제공할 수 있으면 좋겠습니다. 
이들은 Topology에 따라 구분됩니다. 
Continuous system은 각 점을 중심으로 하는 open ball이 neighbor가 되는 topology를 가지고 있고,
Lattice나 complex network는 각 node와 연결된 node들이 neighbor가 될 것입니다.
여러 시스템이 결합된 새 시스템의 neighbor는 product topology로 neighbor를 정의할 수 있습니다.

#### Point, Neighbor
Continuous system에서 neighbor를 정의하는게 무슨 의미가 있냐고 물을 수 있습니다.
이는 다른 시스템에서 neighbor가 어떤 의미를 가지는지 생각해보면 명확합니다.
이산적 시스템에서 neighbor는 '한 번에 이동할 수 있는 점'을 지칭합니다.
연속적인 시스템에서도 같은 의미로 neighbor를 정의할 수 있고, 
'오차범위 내의 근사가 될 수 있도록하는 입자의 이동의 최대 거리'로 사용할 수 있을 것입니다. 

따라서 System을 구성하기 위해 필요한 것은 '점'과 '이웃'입니다.
필요한 구성은 아래와 같을 것입니다. 
Neighbor가 제공해야하는 기능은 차차 생각해보도록 합시다.

```rust
trait Neighbor{
  ???
}

trait Point{
  fn neighbor(&self) -> Neighbor;
  fn displacement(&self, other : &Self) -> Self;
  fn distance(&self, other : &Self) -> F;
}
```

실제 시뮬레이션에서는 아래와 같은 pseudocode 처럼 작동할 것입니다.
``` rust
let nb = point.neighbor();
let new_pos = ... ;
if new_pos in nb{
  point.move_to(new_pos);
}
```

#### Product Topology
시스템간의 결합은 어떻게 정의할 수 있을까 고민해봅니다.
서로 다른 구조들이 tuple로 묶여 다시 Neighbor와 Point를 구성해야하는데,
이 부분은 아래와 같은 형태로 generic type을 이용해 구현할 수 있으리라 생각이 듭니다.
구조가 간단하니 4-5개까지의 tuple에 대해서 macro를 통해 구현할 수 있을 것으로 보입니다. 

``` rust
impl<T : Point> Point for (T, T){
  fn neighbor(&self) -> neighbor{
    (self.0.neighbor(), self.1.neighbor())
  }
}
```

### Boundary Condition
Boundary condition은 경계면과 경계면의 특성 두 가지로 구성됩니다.
경계면은 위치를 정의역으로 하는 실함수 $f(p)$가 0이 되는 hyper-surface로 정의될 것이고,
경계면의 특성은 흡수 경계, 반사 경계, 반복 경계, 산란 경계 등 다양하게 주어질 수 있습니다.

실제 시스템에서는 여러 boundary가 동시에 존재할 수 있습니다.
최근 제가 주로 연구하고 있는 목표 탐색 문제에서는 시스템 경계는 반사 경계,
입자들이 찾고자하는 목표는 흡수 경계로 설정되어있습니다.

실제 시뮬레이션에서는 여러 Boundary condition을 정의하고, 
입자들이 이동할 때마다 boundary condition을 확인해 위치를 교정해주거나 시스템에서 제외하는 기능이 필요합니다. 

```rust
trait BoundaryCondition{
  fn inclusion(&self, pos : &Point) -> bool;
  fn correction(&self, pos : &mut Point, move : &Point);
}
```

여기서 correction 함수는 경계에 따라 필요한 인자가 달라질 수 있을 것으로 보입니다.
흡수 경계는 입자를 시스템으로부터 제거하는 작업을 해야하니 입자들의 리스트 정보에 접근해야하고,
반사 경계는 반사해 다시 안으로 튕겨져 들어오는 경로를 계산해야하니 법선 정보가 필요합니다.
반복 경계는 대응되는 반대편 면으로 입자를 보내야하니 해당하는 함수가 정의되어 있어야합니다.
산란 경계의 경우에는 임의의 방향으로 튕겨나가야하기 때문에 난수 생성기가 필요합니다. 
물론 이런 인자들은 구조체 내부에 정의할 수 있도록 하면, 경계조건 trait은 이정도면 충분할 것입니다.

그렇다면 Boundary trait은 시뮬레이션에서 아래와 같이 사용될 것입니다.
``` rust
enum BC{
  ABC,
  PBC,
  RBC,
  SBC,
}

let point = ... ;
let movement = ... ;
let bcs : Vec<BC> = ... ;

for bc in bcs{
  if bc.correction(pos, move){
    break;
  }
}
```
실제 코딩에서도 각각의 경계에 따라 경계조건을 확인하기 때문에 위의 코드가 비효율적인 것은 아닐겁니다. 

### Force computation

그 다음 해야할 일은 force computation입니다.
입자들은 MD에선 Newton 운동방정식을, BD에서는 Langevin 운동방정식을 따라 움직입니다.
\begin{align}
  m \dv{\vb{v}}{t} = \vb{F}, \quad
  m \dv{\vb{v}}{t} = \vb{F}  + \xi
\end{align}
따라서 속도 변화를 추론하기 위한 외력항과 random force를 계산할 수 있어야합니다.

#### External force
외력에는 중력, 자기장과 같은 global potential 혹은 저항력처럼 단일 입자의 운동 정보로만 구할 수 있는 경우와, 두 입자간 상호작용처럼 둘의 상대적인 운동 정보로 구할 수 있는 경우, 두 가지로 나뉩니다. 
그리고 random force의 경우에는 Gaussian을 따르는 white noise도 있을 수 있지만,
(force는 아니지만) run and tumble에서 방향을 결정하는 무작위 변수는 균등 분포를 따르고,
또 levy walk의 경우에는 이동시간이나 속도가 power-law distribution을 따르도록 할 수 있습니다. 
따라서 force computation에서 정의되어야 하는 함수들은
```rust
fn single_ptl_force(ptl : &Particle) -> Force;
fn double_ptl_force(ptl1 : &Particle, ptl2 : &Particle) -> Force;
```
처럼 쓰일 것입니다.  
문제는 이러한 함수들을 모두 미리 정의해놓을 수는 없단 점이고, 
시뮬레이션 코드를 짜는 단계에서 필요에 따른 force항을 정의하게 될 것입니다. 
따라서 force trait은 함수들보다는 결과물(위 코드의 Force)의 특성에 집중하는 것이 중요하다 생각합니다.
Force는 이후 Movement를 계산하는 도구로써 역할하는데, 연속적인 공간에서는 위치벡터와 동일하게 벡터일 수 있지만, network 등에서는 위치벡터와 얼마든지 다른 구조를 가질 수 있습니다.
이는 Movement에서 함수의 인자로 사용될 것입니다.

#### Random force
random force에 필요한 분포들은
```rust
fn uniform(min : f64, max : f64) -> f64;
fn gaussian(mean : f64, stddev : f64) -> f64; 
fn power_law(coeff : f64, exponent : f64) -> f64; 
```
등의 기능들이 필요할 것이라 예상됩니다.

#### Force trait
이렇게 force항이 여러 개로 나뉠 수 있으니 Force는
``` rust
trait Force{};
impl(F : Force) Force for (F, F){};
```
꼴로 정리해 쓸 수 있도록 해야겠습니다. 

### Movement

Movement는 기본적으로 위치정보, 속도정보, 시간간격, 작용한 힘 들을 모두 종합해 입자의 운동정보를 고치는 작업입니다.
이는 근본적으로 Newton 운동방정식이나 Langevin 운동방정식과 같은 미분방정식을 근사하는 과정인데,
같은 방정식이라도 근사의 방법론이 달라질 수 있습니다.    
어떤 방법론으로 근사할지 선택할 수 있도록 여러 함수들을 구현해놓는 것이 필요할 것입니다.   

그 이전에 위치 정보, 속도 정보, 시간 간격, 힘에 해당하는 trait과 movement trait이 가져야할 기능들을 따져 구현해봅시다. 

#### State trait
먼저 위치 정보와 속도 정보를 담는 State trait부터 살펴봅시다. 
State trait은 Movement trait 구조를 인자로 받아 상태를 새로 고치는 기능이 필요합니다. 
```rust
trait State{
  fn move_to(&mut self, mov : &Movement); 
}
```

#### Time iterator trait
시간 간격은 iterator로써 구현되면 좋습니다.
시뮬레이션에서는 경우에 따라 초반부 변화가 중요할 수 있습니다.
그런 경우에는 초반에 매우 작은 시간간격으로 시뮬레이션하다가, 점차 시간간격을 키워나가는 방식으로 시뮬레이션 할 수 있습니다. 
이처럼 시간간격이 변화하는 방법에 따라 지금의 시점과 지난 시점과의 간격을 제공하는 iterator로서 시간간격을 정의할 수 있습니다. 
```rust 
impl Iterator for TimeIterator{
    type Item = f64;

    fn next(&mut self) -> Option<Self::Item>{
        ...
    }
}
```
이 방법은 상황에 따라 다양한 구조로 쓰여질 수 있습니다.
예를 들어 run and tumble하는 입자들 같은 경우에는 무작위적으로 주어진 시간동안 직진하는 운동을 할텐데, 이런 경우 time iterator는 입자가 운동방향을 바꾸는 가장 가까운 시점을 출력하는 방법으로 시뮬레이션 할 수 있습니다. 
```rust
impl Iterator for MinTimeIterator{
  type Item = f64;
  
  fn next(&mut self) -> Option<Self::Item>{
    return Some(self.times.pop());
  }
}
```

#### Movement trait

이런 관점에서 Movement trait은 아래와 같은 기능을 제공할 것입니다.
``` rust
trait Movement{
  fn compute_move(state : &State, dt : T, force : Force) -> Self
}
```

### Data in/out & Analysis
어떻게 보면 이 crate를 기획하게 된 가장 큰 이유 중 하나입니다. 
시뮬레이션의 구조는 비슷한데 매번 새로운 코드인양 복붙해와 구조를 확인해가며 써야한단 점도 낭비였고,
사용성을 위해 여러 기능을 제공할 필요가 있었습니다. 
input argument에 대한 description을 제공하는 기능이나
configuration을 깔끔하게 정리해 데이터 파일에 출력하고 이를 다시 읽을 수 있는 기능,
데이터 형식에 따라 맞는 analysis를 제공하는 기능을 만들어두고 싶습니다.
BD에서는 ensemble이 여러 파일에 나뉘어 저장되어 있을 수 있습니다.  
이 경우, 시뮬레이션 정보를 읽어 같은 시스템에 대한 데이터는 하나로 묶어 계산할 수 있어야합니다. 

원하는 구성은 아래와 같습니다. 
``` rust
fn brief_instruction() -> String;
fn instruction() -> String;
fn export(&self) -> String;
fn from(args : Vec<String>) -> Self;

// main.rs
fn main(){
  let args : Vec<String> = std::env::args();
  
  // 물론 이렇게 하면 외부에서 sys, bcs 등을 쓸 수 없습니다.
  // pseudo code로 이해해주세요.
  let read_argument = || -> Result<(), Error>{
    if args.next() == Some(0){  // for simulation mode
      let sys = System::from(args)?;
      let bcs = Boundary::from(args)?;
      ...  
    } else { // for analysis mode
      let analysis = Analysis::from(args);
      analysis.analyze();
      return;
    }
  } 
  
  if let Err(e) = read_argument(){
    System::brief_instruction();
    Boundary::brief_instruction();
    ...
    
    System::instruction();
    Boundary::brief_instruction;
    ...
    
    return;
  }
  
  // Body
  output.write!(sys.export());
  output.write!(bcs.export());
  ...
}
```
#### Analysis trait

여기서 아직 설명되지 않은 부분은 Analysis trait 부분입니다. 
Analysis trait은 크게 두 가지 일을 합니다.
- Import simulation info : 시뮬레이션 정보를 읽어오는 기능입니다. 이 기능은 여러 데이터 파일을 한 번에 처리할 때 필요한데, 특히 BD의 경우에는 같은 시스템에 대한 다른 ensemble 결과를 여러 데이터 파일을 취합해 처리해야하기 때문에 시뮬레이션 정보를 읽어오는 것뿐 아니라 이들을 hash하는 기능까지 제공할 필요가 있습니다.
- analysis : 그 후에는 이름 그대로 분석 기능을 제공합니다. 여기엔 여러 분석이 존재할 수 있습니다. System 위치 정보를 그대로 읽어올 수도 있을테고, configuration을 바꿔가며 달라지는 결과의 분포를 확인할 수도 있을 것입니다. 결과물이 여러 변수일 수도 있고, 시간에 따라 바뀌는 process의 결과물일 수도 있겠죠. 이런 경우의 수들을 모두 종합해 읽어올 수 있으면 좋겠습니다. 

실제 내부 구조는 아래와 같습니다. 
```rust
fn analyze(&self){
  datasets : HashMap<Config, Data> = HashMap::new();
  
  // Read datas
  for file in directories{
    input = open(file, "r");
    config = Config::from(input);
    
    dataset = match (config in datasets){
      true => datasets[config],
      false => {
        let ds = dataset::new();
        datasets.push((config, ds));
        ds
    }
    
    for line in input{
      data = line.parse();
      dataset.push(data);
    }
  }
  
  // Analyze data
  for dataset in datasets.values(){
    ...  // depend on data form 
  }
}
```

따라서 Analysis trait에는 세 가지가 미리 정해져있어야합니다.
- Config : 시스템을 구분할 configuration이 어떻게 정의되어있는지.
- Data : 어떤 형태의 데이터를 분석하려고 하는지
- Method : 데이터를 어떤 식으로 분석할 것인지

#### Config trait
Config는 시뮬레이션을 만드는 시점에 정의될 것이고,
Data나 Method는 미리 정의되어있어야 합니다.
그럼으로 우리에게 필요한 것은 Config를 쉽게 정의할 수 있도록 하는 가이드와
다양한 data와 method를 미리 구현해놓는 것이라 할 수 있겠습니다.

Config는 시스템 구성요소들의 parameter들을 성분으로 가지는 tuple structure를 띄어야 하고, 
이를 HashMap에 사용하려면 필히 Hash trait을 정의해줘야합니다. 
즉, 우리는 Data file에 적힌 meta data로부터 특정한 tuple structure를 구성하는 함수가 필요하고, 이에 대한 Hash까지 만들어줄 macro가 필요합니다. 

이를 위해 Config Trait을 정의하는 방법을 생각해봅시다.
``` rust
trait Config{
  fn export(&self) -> String;
  fn from_arg(args : Vec<String>) -> Self;
  fn from_string(strings : Vec<String>) -> Self;
  fn hash()
}
```
위와 같은 trait을 정의하고 아래와 같은 code를 macro로 자동화하면 간단하지 않겠나 싶습니다. 
``` rust
impl<T : Config> Config for (T, T){
  ...
}
```

#### Data structure, Method trait
그리고 Data는 몇 개 변수로 이뤄져있는지, 혹은 process와 같이 변수의 나열로 이뤄져있는지 등을 나누고, 그에 맞는 method를 정의해주면 되겠습니다.
```rust
struct SingleVariable;
struct VariableVec;
struct Process;

trait Histogram{
  fn draw_histogram();
}

impl Histogram for SingleVariable{
  fn draw_histogram();
}
```


### Simulation methods
시뮬레이션을 하는 과정에서 속도를 높일 수 있는 많은 방법들이 있습니다. 
그 중 몇 개는 필히 만들어두고 싶습니다.

#### Verlet list
Verlet list는 Short range force computation을 가속할 때 주로 쓰이는 방법입니다. 
먼 거리의 입자들에게는 어차피 상호작용의 세기가 매우 미미할 것이기 때문에 일정 거리 이하로 가까운 입자들만 추려 상호작용을 계산할 수 있도록 합니다. 
입자들의 상호작용을 무시할 수 있는 cut-off distance를 $R_c$라 하고, 
시뮬레이션 한 스텝동안 입자들이 움직이는 최대 거리 $d$,
Verlet list를 수정하는 주기를 $n$ step이라 했을 때 Verlet list은 입자 주변의 $R_c + 2nd$ 거리에 있는 입자들로 구성하면 됩니다. 
아마 구성은 아래와 같을 것입니다.
``` rust
trait<T : Point> VerletList for LinkedList<T>{
  fn renew(&self, verlet : Vec<LinkedList<T>>, rc : f64){
    verlet.clean();
    for (idx1, agent1, idx2, agent2) in self.pair_enumerate(){
      if agent1.distance(agent2) < rc{
        verlet[idx1].push(idx2);
        verlet[idx2].push(idx1);
      }
    }
  }
}

// Body
for idx, agent in agent_list.enumerate(){
  for agent2 in verlet[idx]{
    // force computation
  }
}
```

#### Cell method
Verlet list와 비슷한 컨셉으로 시뮬레이션을 가속하는 방법입니다.
이건 입자들을 기준으로 하는 것이 아니라 시스템을 잘게 쪼갠 cell을 기준으로 하는겁니다. 
Cell의 크기를 $R_c$의 2배 정도로 하고, 거리 계산을 할 때는 같은 cell에 있는 입자들과, 인접한 cell들에 있는 입자들끼리만 고려하는 방식으로 먼 거리의 입자들을 생략하는 방식입니다.
이 경우, cell에 속해있는 입자들을 renew하는 주기 $n$을 $R_c > 2 n d$ 조건을 만족하도록 잡아주면 됩니다.
``` rust
trait<T : Point> CellList for LinkedList<T>{
  fn renew(&self, cells : Vec<LinkedList<T>>, rc : f64){
    cells.clean();
    for agent in self.iter(){
      cell_idx = agent.cell();
      cells[cell_idx].push(agent);
    }
  }
}

// Body
for cell in cell_list{
  for agent in cell{
    for other_agent in cell{
      // force computation
    }
    
    for other_agent in adjacent_cell{
      // force computation
    }
  }
}
```

#### Higher order approximation
Newton의 운동방정식을 가장 간단히 근사하는 방법은 first order approximation일 것입니다.
\begin{align}
  x(t + \dd t) &= x(t) + v(t + \dd t) \dd t + \frac{1}{2} \frac{F(x,t)}{m} \dd t^2, \newline
  v(t + \dd t) &= v(t) + \frac{F(x,t)}{m} \dd t 
\end{align}
Overdamped Langevin equation
\begin{align}
  \dv{x}{t} = - \mu F(x,t) + \sqrt{2 D} \xi
\end{align}
의 1차 근사해는 아래처럼 쓰이구요.
\begin{align}
  x(t + \dd t) = x(t) - \mu F(x, t) \dd t + \sqrt{2 D \dd t }Z, \quad
  Z \sim N(0,1)
\end{align}

하지만 이들의 근사식은 이런 것만 있는 것은 아닙니다.
보다 Higher order로 결과를 근사하는 방법들이 얼마든지 있을 수 있습니다.
이런 경우까지 충분히 포괄해 문제를 해결할 수 있으면 좋겠습니다. 

### Macro

마지막으로 새 crate를 개발하고자하는 목표 중 다른 하나인 macro 구성을 이야기해봅시다.
지난 crate는 그 구조가 너무 조악해 simulation을 새로 구성하는데 너무 많은 손이 갔습니다.
특히, agent에 새로운 기능을 추가하는 것이 그렇습니다.
예를 들어 Interacting Brownian agent가 상호작용하다가 결합하게 되는 경우를 생각해봅시다.
시뮬레이션에서 이들간의 Merge 기능을 추가하고 싶을 때, 새 기능을 추가하기 위해서는 많은 품이 듭니다. 
그래서 지금 원하기로는 procedural macro를 정의해 아래와 같이 간단히 기능을 추가할 수 있으면 좋겠습니다. 
``` rust
[#derive(Brownian(f64), Merge)]
struct Agent{
  ...
}
```

## Docs.rs 
이 모든 내용을 개발하는 과정에서 충분히 comment를 달아 docs.rs 문서를 추후에 다시 만들 일을 줄일 수 있으면 좋겠습니다.
[매뉴얼](https://doc.rust-lang.org/rustdoc/what-is-rustdoc.html)과 [블로그](https://blog.guillaume-gomez.fr/articles/2020-03-12+Guide+on+how+to+write+documentation+for+a+Rust+crate)를 참고하여 잘 작성해봅시다. 

