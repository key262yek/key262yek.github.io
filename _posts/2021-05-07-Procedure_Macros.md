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

제 Github에는 [rts](https://github.com/key262yek/rts)[^1]라는 repository가 있습니다.
Random Target Search 시뮬레이션을 위해서 다양한 System, Target, Searcher들을 미리 구현해두고, 나아가 Time iterator를 constant time step, exponential time step 등의 조건도 선택해 시뮬레이션 할 수 있도록 하고 있습니다.
규칙에 맞춰 코드를 작성하면 시뮬레이션에 필요한 input parameter들도 알려주고, 데이터 값을 읽어 분석하는 것도 한번에 구현할 수 있습니다.

[^1]: [https://github.com/key262yek/rts](https://github.com/key262yek/rts)

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
setup_simulation!(args, 15, 1, TimeAnalysis, 
        "RTS_N_PTL_EXP_SEARCHER", 
        dataset, SimulationData, sys_arg, ContCircSystem, 
        target_arg, ContBulkTarget, searcher_arg, ContPassiveLJSearcher, 
        time_arg, ExponentialStep, sim_arg, VariableSimulation);
~~~

위 macro는 시뮬레이션에서 필요한 argument를 실제로 parsing하는 macro입니다.
argument들은 system, target, searcher, time, simulation 정보들이 바뀌면 알아서 바뀌게 되는데, 필요한 정보가 딱 맞춰 입력되지 않으면 에러를 출력하며 필요 정보에 대한 리스트를 제공해줍니다.

또 TimeAnalysis는 이 시뮬레이션이 특정한 시간 하나를 측정하고 이를 분석하는 시뮬레이션이란 의미라, 그에 맞는 Analysis 함수를 미리 설정해줍니다.
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
simulation!("RTS_N_PTL_EXP_SEARCHER", TimeAnalysis, 
        ContCircSystem, ContBulkTarget, ContPassiveLJSearcher, 
        ExponentialStep, VariableSimulation);
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

Procedure macros에 대한 설명은 [the book](https://doc.rust-lang.org/reference/procedural-macros.html)[^2]과 [blog](https://taeguk2.blogspot.com/2019/02/rust-procedural-macros-by-example.html)[^3]를 참고하면 되는데, 여기서는 Procedural macro 중에서 **Function-like macro**를 이용할 것입니다.

[^2]: [https://doc.rust-lang.org/reference/procedural-macros.html](https://doc.rust-lang.org/reference/procedural-macros.html)
[^3]: [https://taeguk2.blogspot.com/2019/02/rust-procedural-macros-by-example.html](https://taeguk2.blogspot.com/2019/02/rust-procedural-macros-by-example.html)

Rust의 일반 함수처럼 macro를 짤 수 있는 방법인데, compile time에서 함수가 호출되어 컴파일될 코드를 작성해주는 macro입니다.
지금의 계획은 identifier를 입력하면, 각 매크로에 넣을 token tree를 출력해주는 macro와 
이 token tree들을 받아 실제로 코드로 만들어줄 macro, 두 종류의 macro를 작성하고자 합니다.

#### create new crate

먼저 해야할 일은 새 library crate를 만드는 것입니다.
Procedural macro는 macro를 사용할 crate와 다른 crate에서 선언되어야 하는데요.
이는 본 crate를 compile하면서 macro를 호출할 수 있도록, 먼저 compile되어있어야 하기 때문으로 보입니다.
따라서 새로운 library crate [*rts_proc*](https://github.com/key262yek/rts_proc)[^4]를 만들었습니다.
그리고 *Cargo.toml* 파일에 아래와 같이 procedural macro를 위한 library임을 명시해줍니다.
또 많이 사용하는 crate들을 dependency에 서술해줍니다.

~~~ rust
// Cargo.toml

[lib]
proc-macro=true

[dependencies]
proc-macro2="1.0"
proc-quote="0.4"
syn="1.0"
~~~

[^4]: [https://github.com/key262yek/rts_proc](https://github.com/key262yek/rts_proc)


#### Parsing identifier

그 다음엔 identifier를 받아 parsing하는 함수를 사용하는 방법을 알아야합니다.
procedural macro 내부에서 token들은 주로 [syn](https://docs.rs/syn/1.0.72/syn/index.html)[^5] crate를 통해 다룹니다.
syn crate을 통해 parse trait을 사용할 수 있기 때문에 이를 이용해 input identifier를 parsing 해봅니다.

[^5]: ["https://docs.rs/syn/1.0.72/syn/index.html"](https://docs.rs/syn/1.0.72/syn/index.html)

~~~ rust
#[proc_macro]
pub fn test_input(input: proc_macro::TokenStream) -> proc_macro::TokenStream {
    let ident = syn::parse_macro_input!(input as proc_macro2::Ident);

    let tokens = proc_quote::quote!{
        println!(stringify!(#ident));
    };
    tokens.into()
}
~~~

이렇게 하면 input의 첫번째 identifier를 parsing할 수 있습니다.
위 함수는 그렇게 parsing한 함수를 출력해주는 함수입니다.
실행 코드와 출력사진은 아래와 같습니다. 
~~~ rust
use rts_proc::print_input;

fn main(){
    test_input!(Hello_world);
    test_input!(It_work);
}
~~~
![실행파일1](/assets/images/20210507_proc_macro_1.png)

#### Parsing complex input

하지만 한 개의 identifier만 parsing하는 것으로는 우리가 원하는 복잡한 구조의 macro를 만드는데 부족합니다. 
우리가 원하는 것은 아래와 같은 매크로를 선언하는 것이므로, parsing도 이에 맞춰 정의해두는 것이 좋습니다. 
~~~ rust
simulation!("RTS_N_PTL_EXP_SEARCHER", TimeAnalysis, 
        ContCircSystem, ContBulkTarget, ContPassiveLJSearcher, 
        ExponentialStep, VariableSimulation);
~~~

먼저 argument를 parsing할 구조체부터 선언합니다.
~~~ rust
#[derive(Debug)]
struct ParsedArguments{
    prefix      : proc_macro2::Literal,
    setups      : Vec<proc_macro2::Ident>, 
}
~~~
`prefix`는 파일들 앞에 붙을 접두어를 선언해주는 것이고, 나머지는 모두 `setups`에 들어가게 될 것입니다. 

parsing은 `syn::parse::Parse` trait을 이용하면 비교적 간단합니다. 

~~~  rust
impl syn::parse::Parse for ParsedArguments {
    fn parse(input: syn::parse::ParseStream) -> syn::Result<Self> {
        let mut parsed_args = ParsedArguments {
            prefix      : proc_macro2::Literal::string("RTS_GENERAL_"),
            setups      : vec![],
        };

        if !input.is_empty() {
            parsed_args.prefix = match input.parse() {
                Ok(prefix) => prefix,
                Err(err) => return Err(syn::Error::new(err.span(), format!(
                    "The first token must be an Literal"))),
            };

            // 그 다음부터는 comma (,) 와 인자 타입을 의미하는 identifier 가 반복해서 와야한다.
            let mut i = 1;
            while !input.is_empty() {
                // comma 인지 체크
                match input.parse::<syn::token::Comma>() {
                    Ok(_) => {},
                    Err(err) => return Err(syn::Error::new(err.span(), format!(
                        "The token {} must be a comma.", i + 1))),
                };
                i += 1;

                if input.is_empty() {
                    break;
                }

                // Identifier 인지 체크
                let type_name = match input.parse::<syn::Ident>() {
                    Ok(type_name) => type_name,
                    Err(err) => return Err(syn::Error::new(err.span(), format!(
                        "The token {} must be an identifier.", i + 1))),
                };
                parsed_args.setups.push(type_name);
                i += 1;
            }
        }
        Ok(parsed_args)
    }
}
~~~
기본적인 코드의 틀은 [taeguk님의 블로그](https://www.secmem.org/blog/2019/02/10/rust-procedural-macros-by-example/)[^6]

[^6]: ["https://www.secmem.org/blog/2019/02/10/rust-procedural-macros-by-example/"](https://www.secmem.org/blog/2019/02/10/rust-procedural-macros-by-example/)

제대로 parsing이 이뤄지는지 체크해봅니다.
~~~ rust
// procedural macro library
#[proc_macro]
pub fn test_parse(input: proc_macro::TokenStream) -> proc_macro::TokenStream{
    let ident = syn::parse_macro_input!(input as ParsedArguments);
    
    let prefix = ident.prefix;
    let setups = &ident.setups;

    let tokens = proc_quote::quote!{
        println!("{:?}", #prefix);
        #(println!(stringify!(#setups));)*
    };
    tokens.into()
}

// source
fn main(){
    test_parse!("RTS_N_PTL_EXP_SEARCHER", TimeAnalysis, 
        ContCircSystem, ContBulkTarget, ContPassiveLJSearcher, 
        ExponentialStep, VariableSimulation);
}
~~~
![실행파일3](/assets/images/20210507_proc_macro_3.png)

#### Matching identifier

이젠 parsing한 identifier를 matching해 case by case별로 token tree를 다르게 만들어줘야합니다. 
이를 위해 identifier의 match 코드를 확인해봅니다. 

~~~ rust
fn match_ident(ident : proc_macro2::Ident) -> proc_macro::TokenStream{
    match ident.to_string().as_str() {
        "Should_matched" => {
            let tt = syn::Ident::new("Matched", proc_macro2::Span::call_site());
            let tokens = proc_quote::quote!{
                println!(stringify!(#tt));
            };
            tokens.into()
        }
        _ => {
            let tokens = proc_quote::quote!{
                println!(stringify!(#ident));
            };
            tokens.into()
        }
    }
}
~~~

proc_macro2에 정의된 Identifier 구조는 내부에 inner라는 field가 있어 unhygienic name을 저장하고 있지만, private하게 선언되어있어 이름을 string으로 받아 matching 하는 수밖에 없습니다.

~~~ rust
// procedural macro library

#[proc_macro]
pub fn test_match(input: proc_macro::TokenStream) -> proc_macro::TokenStream {
    let dataset = syn::parse_macro_input!(input as proc_macro2::Ident);
    match_ident(dataset)
}

// source
use rts_proc::test_match;

fn main(){
    test_match!(Hello_world);
    test_match!(It_work);
    test_match!(Should_matched);
}


~~~
![실행파일2](/assets/images/20210507_proc_macro_2.png)

#### Construct Token Tree

그 다음 해야할 일은 받아낸 identifier와 match를 이용해 token tree를 만드는 것입니다. 
`ContCircSystem`을 argument로 보내면, `ContCircSystem, sys_arg, ContCircSystemArguments, [sys_size, f64, dim, usize]`가 결과로 나오는 함수가 필요합니다. 
이는 위에서도 몇번 활용한 `proc_quote::quote!`를 활용하면 편합니다. 

~~~ rust
fn ident_to_dataset(ident : proc_macro2::Ident) -> proc_macro2::TokenStream{
    match ident.to_string().as_str() {
        // System Types
        "ContCircSystem" => {
            let tokens = proc_quote::quote!{
                ContCircSystem, sys_arg, ContCircSystemArguments, [sys_size, f64, dim, usize]
            };
            tokens.into()
        },
        ...
        "ProcessSimulation" => {
            let tokens = proc_quote::quote!{
                ProcessSimulation, sim_arg, ProcessSimulationArguments, [period, f64, idx_set, usize]
            };
            tokens.into()
        },
        _ => {
            let tokens = proc_quote::quote!{
                #ident
            };
            tokens.into()
        }
    }
}
~~~

이 코드는 곧바로 내용을 확인하기 어렵긴한데, 잘 작동합니다. 
이를 이용해 대부분의 macro를 아래와 같이 고쳐쓸 수 있습니다. 

~~~ rust
pub fn proc_construct_dataset(&self) -> proc_macro2::TokenStream{
    let tt_sys = ident_to_dataset(&self.setups[1]);
    let tt_target = ident_to_dataset(&self.setups[2]);
    let tt_searcher = ident_to_dataset(&self.setups[3]);
    let tt_time = ident_to_dataset(&self.setups[4]);
    let tt_sim = ident_to_dataset(&self.setups[5]);

    let tokens = proc_quote::quote!{
        construct_dataset!(SimulationData,
        #tt_sys; #tt_target; #tt_searcher; #tt_time; {#tt_sim});
    };
    tokens.into()
}

pub fn proc_setup_simulation(&self) -> proc_macro2::TokenStream{
    let prefix = self.prefix.clone();
    let id_analysis = self.setups[0].clone();
    let id_sys = self.setups[1].clone();
    let id_target = self.setups[2].clone();
    let id_searcher = self.setups[3].clone();
    let id_time = self.setups[4].clone();
    let id_sim = self.setups[5].clone();

    let tokens = proc_quote::quote!{
        setup_simulation!(args, 15, 1, #id_analysis, #prefix,
            dataset, SimulationData, sys_arg, #id_sys,
            target_arg, #id_target, searcher_arg, #id_searcher,
            time_arg, #id_time, sim_arg, #id_sim);
    };
    tokens.into()
}

pub fn proc_export_simulation_info(&self) -> proc_macro2::TokenStream{
    let prefix = self.prefix.clone();
    let id_sys = self.setups[1].clone();
    let id_target = self.setups[2].clone();
    let id_searcher = self.setups[3].clone();
    let id_time = self.setups[4].clone();
    let id_sim = self.setups[5].clone();

    let tokens = proc_quote::quote!{
        export_simulation_info!(dataset, output_dir, writer, WIDTH,
            #prefix,
            #id_sys, sys, sys_arg,
            #id_target, target, target_arg,
            #id_searcher, vec_searchers, searcher_arg,
            #id_time, timeiter, time_arg,
            #id_sim, simulation, sim_arg);
    };
    tokens.into()
}
~~~

#### Declare variables

여기까지 만들어야하는가 가장 고민되는 지점이긴 합니다.  
변수 선언을 macro에 넣어놓으면 어떤 변수가 있는지 코드를 보고서는 확인할 수 없습니다.
하지만 현재도 시뮬레이션이 작동하기 위해서 어떤 argument를 입력해야하는지도 코드를 돌려봐야알고,  과정에서 정의되는 변수들을 어차피 확인할 수 있게 됩니다. 
따라서 개인적인 편의를 위해 이 부분도 macro 에 넣기로 결정했습니다.

기본적인 방법은 동일합니다.
identifier에 대응되는 field명들을 list로 받고, 다시 독립적인 변수로 선언해주면 될 것입니다.
먼저 identifer로부터 list of field를 만드는 함수부터 만들어줍니다. 

~~~ rust
fn ident_to_variables(ident : proc_macro2::Ident) -> Vec<proc_macro2::Ident>{
    fn string_to_ident(name : &str) -> proc_macro2::Ident{
        syn::Ident::new(name, proc_macro2::Span::call_site())
    }

    let mut vec : Vec<proc_macro2::Ident> = Vec::new();

    match ident.to_string().as_str() {
        "ContCircSystem" |
        "ContCubicSystem" => {
            vec.push(string_to_ident("sys_size"));
            vec.push(string_to_ident("dim"));
        },
        "ContCylindricalSystem" => {
            vec.push(string_to_ident("sys_radius"));
            vec.push(string_to_ident("sys_length"));
            vec.push(string_to_ident("dim"));
        },

        ...

        _ => {}
    }
    vec
}
~~~

그 후 동일한 이름의 identifier에 argument들의 field들을 대응시켜줍니다.
~~~ rust 
let id_sys = self.setups[1].clone();
let var_sys = ident_to_variables(id_sys);

let id_target = self.setups[2].clone();
let var_target = ident_to_variables(id_target);

let id_searcher = self.setups[3].clone();
let var_searcher = ident_to_variables(id_searcher);

let id_time = self.setups[4].clone();
let var_time = ident_to_variables(id_time);

let id_sim = self.setups[5].clone();
let var_sim = ident_to_variables(id_sim);

let tokens = proc_quote::quote!{
    #(let #var_sys = sys_arg.#var_sys.clone();)*
    #(let #var_target = target_arg.#var_target.clone();)*
    #(let #var_searcher = searcher_arg.#var_searcher.clone();)*
    #(let #var_time = time_arg.#var_time.clone();)*
    #(let #var_sim = sim_arg.#var_sim.clone();)*
};
tokens.into()
~~~

최종적으로 이런 구조를 모두 모아 macro를 완성했습니다.
~~~ rust 
#[proc_macro]
pub fn simulation(input: proc_macro::TokenStream) -> proc_macro::TokenStream{
    let ident = syn::parse_macro_input!(input as ParsedArguments);

    let token_cd = ident.proc_construct_dataset();
    let token_setup = ident.proc_setup_simulation();
    let token_var = ident.proc_variable_declare();
    let token_export = ident.proc_export_simulation_info();

    let tokens = proc_quote::quote!{
        #token_cd
        #token_setup
        #token_var
        #token_export
    };
    tokens.into()
}
~~~

### Closing

물론 macro를 외부에서 사용하도록 crate를 구성했단 면에서 제작자 외에는 (심지어는 제작자도 시간이 지나면 어떨지...) 사용성이 매우 떨어지는 crate임에는 분명한 것 같습니다.
개별 시뮬레이션이 아닌 좀 더 다양한 경우를 포괄할 수 있는 구조로 crate를 짜고자 했는데, 편의성을 담보하기 위해 사용했던 macro가 결국 끝에와서 이렇게 발목을 잡네요.
첫 경험이니 이정도에 만족하지만, 체계적인 프로젝트에 참여해 큰 구조를 계획하는 방법을 배울 필요성이 있는 것 같습니다.

어쨌든 매우 복잡한 코드를 짜버렸고, 시간 지나면 다시 못쓰게 될 것 같으니 빠른 시일내로 전반적인 comment 작업을 시작해야할 것 같습니다.
이왕 comment 작업하는 김에 crate publish에 대해서도 공부해보고. 

## References
