---
title: "Resolve a panic when clap's get_one() function downcast f64"
date: 2022-01-30-T14:15:30-14:30
categories:
  - Programmings
tags:
  - Rust
  - Clap
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

Rust에는 [Clap](https://docs.rs/clap/latest/clap/index.html)이라는 crate가 있어 Argument를 보다 깔끔하게 정의하고, help document를 만들어주며 parsingRkwㅣ 도와준다. 
Argument parsing의 중심에는 `get_one()`이란 함수가 있는데, input string에서 boolean, usize 등의 자료형으로 변환하는데 도움을 준다. 
하지만 내 코드에서는 이유를 모르겠는 panic이 하나 나타난다. 
이 글은 바로 그 에러의 원인을 찾고 해결하는 과정을 담아본다. 

### Minimal code for problem
아래와 같은 코드를 작성했다 하자. 아래 코드는 이 프로그램의 이름을 "test"라 설정하고, "mass"라는 argument 하나를 받는데 그를 지칭하는 short flag가 'm'이라 두었다. 
그리고 다음 줄에서 입력 받은 argument를 f64 자료형으로 parsing했다.  

```rust 
use clap::builder::Command;
use clap::{Arg, ArgMatches};

fn main(){
  let matches = Command::new("test")
                .arg(Arg::new("mass")
                    .short('m')
                    .takes_value(true))
                .get_matches()
  let mass: f64 = *matches.get_one::<f64>("mass").unwrap();  
}
```
이 코드는 전혀 문제없이 컴파일 된다. 
하지만 문제는 runtime에서 생기는데, 아래와 같이 argument를 입력하게 되면 panic이 발생한다. 
```rust 
./minimal -m 1.0
```
에러 메세지는 "Mismatch between definition and access of `mass`. Could not downcast to f64, need to downcast to alloc::string::String" 와 같이 뜬다. 

### Backtrace panic
위의 panic이 어떤 함수에서 기인한 것인지 확인해보자. 
먼저 위의 메세지는 [clap::parser::MatchesError::unwrap()](https://docs.rs/clap/latest/src/clap/parser/error.rs.html) 함수의 결과물이다. 
`try_get_one()` 함수가 Ok값을 주지 않아 에러 메세지와 함께 panic 한 것이다. 

try_get_one() 함수의 구조는 아래와 같다. 
```rust
impl ArgMatches {
    pub fn try_get_one<T: Any + Clone + Send + Sync + 'static>(
        &self,
        id: &str,
    ) -> Result<Option<&T>, MatchesError> {
        let id = Id::from(id);
        let arg = self.try_get_arg_t::<T>(&id)?;
        let value = match arg.and_then(|a| a.first()) {
            Some(value) => value,
            None => {
                return Ok(None);
            }
        };
        Ok(value
            .downcast_ref::<T>()
            .map(Some)
            .expect(INTERNAL_ERROR_MSG)) // enforced by `try_get_arg_t`
    }
}
```
여기서 Error가 전달되는 부분은 `try_get_arg_t()`뿐이다. 
```rust 
impl ArgMatches {
    #[inline]
    fn try_get_arg(&self, arg: &Id) -> Result<Option<&MatchedArg>, MatchesError> {
        self.verify_arg(arg)?;
        Ok(self.args.get(arg))
    }

    #[inline]
    fn try_get_arg_t<T: Any + Send + Sync + 'static>(
        &self,
        arg: &Id,
    ) -> Result<Option<&MatchedArg>, MatchesError> {
        let arg = match self.try_get_arg(arg)? {
            Some(arg) => arg,
            None => {
                return Ok(None);
            }
        };
        self.verify_arg_t::<T>(arg)?;
        Ok(Some(arg))
    }
}
``` 
여기서 error가 나올 수 있는 영역은 `verify_arg()`함수와 `verify_arg_t()`함수. 
```rust 
impl ArgMatches {
    #[inline]
    fn verify_arg(&self, _arg: &Id) -> Result<(), MatchesError> {
        #[cfg(debug_assertions)]
        {
            if self.disable_asserts || *_arg == Id::empty_hash() || self.valid_args.contains(_arg) {
            } else if self.valid_subcommands.contains(_arg) {
                debug!(
                    "Subcommand `{:?}` used where an argument or group name was expected.",
                    _arg
                );
                return Err(MatchesError::UnknownArgument {});
            } else {
                debug!(
                    "`{:?}` is not an id of an argument or a group.\n\
                     Make sure you're using the name of the argument itself \
                     and not the name of short or long flags.",
                    _arg
                );
                return Err(MatchesError::UnknownArgument {});
            }
        }
        Ok(())
    }

    fn verify_arg_t<T: Any + Send + Sync + 'static>(
        &self,
        arg: &MatchedArg,
    ) -> Result<(), MatchesError> {
        let expected = AnyValueId::of::<T>();
        let actual = arg.infer_type_id(expected);
        if expected == actual {
            Ok(())
        } else {
            Err(MatchesError::Downcast { actual, expected })
        }
    }
}
```
DownCast 에러는 `verify_arg_t()`에서 일어나는 것으로 보이고, 세부적으로는 [MatchedArgs::infer_type_id](https://docs.rs/clap/latest/src/clap/parser/matches/matched_arg.rs.html)에서 문제가 생기는 것이다. 
```rust
impl MatchedArg { 
    pub(crate) fn infer_type_id(&self, expected: AnyValueId) -> AnyValueId {
        self.type_id()
            .or_else(|| {
                self.vals_flatten()
                    .map(|v| v.type_id())
                    .find(|actual| *actual != expected)
            })
            .unwrap_or(expected)
    }
}
```
argument의 type을 반환하거나, 여러 argument가 함께 있다면 그 중 expected type과 다른 type을 반환하는 함수인데, 즉 floating point와 관련한 argument는 f64로 인식되는 것이 아니라 String으로 인식된단 것이다. 
Argument의 구조를 확인하기 위해서는 `try_get_arg()` 내부의 `self.args.get(arg)` 부분을 들여다 본다. 
`args` field는 `IndexMap<Id, MatchedArg>` 구조이고, `MatchedArg`의 정의는 아래와 같이 주어진다. 
```rust 
#[derive(Debug, Clone)]
pub(crate) struct MatchedArg {
    occurs: u64,
    source: Option<ValueSource>,
    indices: Vec<usize>,
    type_id: Option<AnyValueId>,
    vals: Vec<Vec<AnyValue>>,
    raw_vals: Vec<Vec<OsString>>,
    ignore_case: bool,
}

impl MatchedArg {
    pub(crate) fn new_arg(arg: &crate::Arg) -> Self {
        let ignore_case = arg.is_ignore_case_set();
        Self {
            occurs: 0,
            source: None,
            indices: Vec::new(),
            type_id: Some(arg.get_value_parser().type_id()),
            vals: Vec::new(),
            raw_vals: Vec::new(),
            ignore_case,
        }
    }
}
```
`type_id`는 `get_value_parser()` 함수에 따라 정해진다. 
그리고 `get_value_parser()`는 argument에 할당된 ValueParser struct를 반환한다. 

### ValueParser in Clap
그렇다면 문제는 아마도 f64를 parsing할 parser는 자동 지정이 아니라 미리 지정해주어야하는 것이 아닐까? 
```rust 
fn main(){
  let matches = Command::new("test")
                .arg(Arg::new("mass")
                    .short('m')
                    .takes_value(true))
                    .value_parser(clap::value_parser!(f64))
                .get_matches()
}
```
위와 같이 코드를 고쳐주면 더이상 같은 문제는 발생하지 않는다. 
그러면 f64에 대응하는 parser는 어떤 것일까?
직접 출력해본 결과 `_AnonymousValueParser(ValueParser::other(f64))`라는 결론을 얻었다. 
즉, clap의 기본 구조에서는 floating point type 들에 대한 parser를 구비해놓지 않았고, 명시적으로 f64 parser를 지정해주는 경우에만 문제 없이 parsing된단 것을 알 수 있다. 


