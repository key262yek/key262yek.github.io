---
title: "Procedural Macros in Rust"
date: 2021-05-08T21:35:30-22:00
categories:
  - Programmings
tags:
  - Rust
  - Procedural Macros
  - Hygienic
  - Eager Macros 
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

제 Github에는 [rts]("https://github.com/key262yek/rts")[^1]라는 repository가 있습니다.
Random Target Search 시뮬레이션을 위해서 다양한 System, Target, Searcher들을 미리 구현해두고, 나아가 Time iterator를 constant time step, exponential time step 등의 조건도 선택해 시뮬레이션 할 수 있도록 하고 있습니다.
규칙에 맞춰 코드를 작성하면 시뮬레이션에 필요한 input parameter들도 알려주고, 데이터 값을 읽어 분석하는 것도 한번에 구현할 수 있습니다.

[^1]: ["https://github.com/key262yek/rts"]("https://github.com/key262yek/rts")

문제는 이 '규칙'이란 것이 매우 복잡하단 것입니다. 
예를 들어, **연속적인 2D 원형 공간**의 중심에 **원형 목표**를 두고, **Lennard Jones 상호작용을 하는 탐색자**들의 목표 탐색을 **exponential time step**으로 시뮬레이션해 **Mean First Passage Time(MFPT)**을 측정, 분석하는 코드를 작성하고자 한다했을 때, 
(원형 공간, 원형 목표, Lennard Jones 탐색자, Exponential Time step, MFPT Analysis)만 선택하면 참 좋을텐데 그렇지 않습니다.

적당히 복잡하면 모르겠는데, 이 코드가 macro로 쓰여있어 가독성이 매우 좋지 않습니다.
얼마나 복잡하자면... 아래와 같은 macro 세 개가 존재합니다.

~~~ rust
construct_dataset!(SimulationData, 
        ContCircSystem, sys_arg, ContCircSystemArguments,
        [sys_size, f64, dim, usize ];
        ContBulkTarget, target_arg, ContBulkTargetArguments,
        [target_size, f64];
        ContPassiveLJSearcher, searcher_arg, ContPassiveLJSearcherArguments,
        [num_searcher, usize, ptl_size, f64, strength, f64];
        ExponentialStep, time_arg, ExponentialStepArguments,
        [dt_min, f64, dt_max, f64, length, usize];
        {VariableSimulation, sim_arg, VariableSimulationArguments,
        [idx_set, usize]});
~~~

위 macro는 시뮬레이션의 데이터 파일들을 분류하기 위한 기준 configuration이 무엇인지 정해주는 macro입니다. 
즉, 시스템 크기, 시스템 차원, 목표 크기, 입자 수, 입자의 크기, 상호작용의 세기, 시뮬레이션의 최소 길이 단위, 최대 길이 단위, 길이 단위가 변화하는 주기가 같다면 같은 시뮬레이션이고, 같은 조건의 시뮬레이션들끼리는 idx_set으로 구분할 수 있도록 되어있단 의미입니다.

~~~ rust
setup_simulation!(args, 15, 1, MFPTAnalysis, 
        "RTS_N_PTL_EXP_SEARCHER", 
        dataset, SimulationData, sys_arg, ContCircSystem, 
        target_arg, ContBulkTarget, searcher_arg, ContPassiveLJSearcher, 
        time_arg, ExponentialStep, sim_arg, VariableSimulation);
~~~

위 macro는 시뮬레이션에서 필요한 argument를 실제로 parsing하는 macro입니다.
argument들은 system, target, searcher, time, simulation 정보들이 바뀌면 알아서 바뀌게 되는데, 필요한 정보가 딱 맞춰 입력되지 않으면 에러를 출력하며 필요 정보에 대한 리스트를 제공해줍니다.

또 MFPTAnalysis는 이 시뮬레이션이 MFPT를 측정하고 이를 분석하는 시뮬레이션이란 의미이고,
따라서 그에 맞는 Analysis 함수를 미리 설정해줍니다.
argument 입력할 때 analysis에 필요한 정보 수에 맞춰 입력하면 알아서 analysis 모드로 코드가 돌아가게 됩니다.

~~~ rust
let sys_size    = sys_arg.sys_size;
let dim         = sys_arg.dim;

let _target_pos  = target_arg.target_pos.clone();
let target_size = target_arg.target_size;

let _mtype       = searcher_arg.mtype;
let _itype       = searcher_arg.itype.clone();
let ptl_size     = searcher_arg.ptl_size;
let strength     = searcher_arg.strength;
let num_searcher = searcher_arg.num_searcher;

let _dt_min          = time_arg.dt_min;
let _dt_max          = time_arg.dt_max;
let _length          = time_arg.length;

let num_ensemble= sim_arg.num_ensemble;
let idx_set     = sim_arg.idx_set;
let seed        = sim_arg.seed;
let output_dir  = sim_arg.output_dir.clone();
~~~

만약 argument들이 시뮬레이션 모드로 잘 입력되었다면, 그 argument들은 각각 somthing_arg 변수들에 저장되고, 이로부터 필요한 정보들을 읽어 시뮬레이션에 써먹을 수 있습니다.

~~~ rust
export_simulation_info!(dataset, output_dir, writer, WIDTH, 
        "RTS_N_PTL_EXP_SEARCHER",
        ContCircSystem, sys, sys_arg,
        ContBulkTarget, target, target_arg,
        ContPassiveLJSearcher, vec_searchers, searcher_arg,
        ExponentialStep, timeiter, time_arg,
        VariableSimulation, simulation, sim_arg);
~~~

마지막으로 이 macro는 argument들로부터 시뮬레이션에 필요한 변수들을 선언하는 macro입니다.
시스템, 목표, 입자들의 리스트, time iterator 등을 정의해주고 본격적으로 시뮬레이션에서 사용될 수 있도록 합니다.

문제는 이 과정들이 시스템 정보들이 결정되면 거의 달라질 부분이 없단 점입니다.
이 모든 것이 하나의 macro로 묶일 수 있단 것이고, 그렇다면 복잡한 방식으로 시뮬레이션 코드를 작성할 필요가 사라지게 됩니다.
원하는 것은 아래와 같은 하나의 macro로 위의 코드들을 하나로 합치는 것입니다.

~~~ rust
simulation!(15, 1, "RTS_N_PTL_EXP_SEARCHER", MFPTAnalysis, 
        ContCircSystem, ContBulkTarget, ContPassiveLJSearcher, 
        ExponentialStep, Simulation);
~~~

### Problems - Hygienic, Eager macro 

이런 macro의 내부에는 ContCircSystem이란 identifier가 들어오면, 이에 대응되는 token tree `ContCircSystem, sys_arg, ContCircSystemArguments, [sys_size, f64, dim, usize]`를 construct_dataset macro에 전달해주는 부분이 들어가야합니다.
이걸 macro로 해결하고 싶지만, 그럴 수 없습니다. 
만약 token_tree 라는 macro를 통해 argument를 적을 수 있도록 하더라도 macro 결과물로 나오는 변수들은 hygienic한 값으로 적힐 것이기 때문에, macro 밖의 코드에서 같은 이름으로 접근할 수 없습니다.
우선 hygienic부터 해결해야겠지만, 이를 해결해도 문제는 남습니다.
왜냐하면 rust의 macro는 기본적으로 lazy-macro이기 때문입니다. 
위에서 생각한 방식대로 코드를 적으면 아래와 같을텐데요.

~~~ rust
construct_dataset!(token_tree!(ContCircSystem), ... );
~~~

우리가 바라는 것은 token_tree macro가 먼저 작동하고 그 다음 construct_dataset macro가 작동하는 것입니다.
이런 작동 방식을 Eager macro라고 하는데, 안타깝게도 rust에서는 몇몇 특수한 경우들을 제외하고는 모두 밖에서부터 macro를 수행하는 lazy macro를 따르고 있습니다. 

두 개의 문제를 모두 해결하지 못하기 때문에, macro 안에 macro를 적는 방법은 해법이 될 수 없습니다.
답이 없는 줄 알았으나, macro 안에서 macro 선언을 할 수 있으면서도, unhygienic token을 임의로 정의할 수 있는 방법이 있었습니다.
그것이 **Procedural Macros**입니다. 

### Solution - Procedural Macros

Procedure macros에 대한 설명은 [the book]("https://doc.rust-lang.org/reference/procedural-macros.html")[^2]과 [blog]("https://taeguk2.blogspot.com/2019/02/rust-procedural-macros-by-example.html")[^3]를 참고하면 되는데, 여기서는 Procedural macro 중에서 **Function-like macro**를 이용할 것입니다.

[^2]: ["https://doc.rust-lang.org/reference/procedural-macros.html"]("https://doc.rust-lang.org/reference/procedural-macros.html")
[^3]: ["https://taeguk2.blogspot.com/2019/02/rust-procedural-macros-by-example.html"]("https://taeguk2.blogspot.com/2019/02/rust-procedural-macros-by-example.html")

Rust의 일반 함수처럼 macro를 짤 수 있는 방법인데, compile time에서 함수가 호출되어 컴파일될 코드를 작성해주는 macro입니다.
지금의 계획은 identifier를 입력하면, 각 매크로에 넣을 token tree를 출력해주는 macro와 
이 token tree들을 받아 실제로 코드로 만들어줄 macro, 두 종류의 macro를 작성하고자 합니다.

#### Construct Token Tree

procedural macro 내부에서 token들을 다루는 방법은 주로 [syn]("https://docs.rs/syn/1.0.72/syn/index.html")[^4] crate를 이용한 방법입니다. 


[^4]: ["https://docs.rs/syn/1.0.72/syn/index.html"]("https://docs.rs/syn/1.0.72/syn/index.html")


## References
