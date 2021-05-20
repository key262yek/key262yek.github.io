---
title: "Publish my own rust crate"
date: 2021-05-08T21:35:30-22:00
categories:
  - Programmings
tags:
  - Rust
  - Publish
toc: true
toc_label: "Table of Contents"
toc_icon: "fas fa-clipboard-list"
toc_sticky: true
---

지난 post를 통해 [rust_rts](https://github.com/key262yek/rust_rts) crate의 구조를 어느정도 완성했습니다.
물론 discrete system이나 boundary에 위치하는 target, active searcher 등 추가적으로 개발해야하는 영역이 많이 남았지만, 큰 틀에서 필요한 부분만 채워넣으면 되는 상황이라 할 수 있습니다.
다만 crate 구조가 조악해 주석을 자세히 달아야지만 시간이 지나도 알맞게 사용할 수 있겠죠.
해서 crate에 주석을 보다 본격적으로 작성하고, 나아가 publish하는 과정까지를 배워보고 정리하고자 합니다. crate publish에 대한 자세한 정보는 [the book](https://doc.rust-lang.org/cargo/reference/publishing.html)[^1] 에서 확인할 수 있습니다.

[^1]:[https://doc.rust-lang.org/cargo/reference/publishing.html](https://doc.rust-lang.org/cargo/reference/publishing.html)

### API token

먼저 해야하는 것은 cargo가 crate.io에 접속해 crate 관리를 할 수 있도록 api login token을 받는 것입니다.
이는 crate.io에 접속해 github 계정으로 로그인한 후 account setting에서 얻을 수 있습니다.
발급받은 token을 아래의 명령어에 입력해 실행해주면, `~/.cargo/credentials` 파일에 cargo가 로그인 정보를 저장하게 됩니다.
~~~ bash
$ cargo login abcdefghijklmnopqrstuvwxyz012345
~~~
이 과정에서 crate.io의 email verification도 같이 진행해주시면 좋습니다.

### Metadata

그 이후에 확인해야 하는 부분은 crate에 대한 metadata가 잘 적혀있는지 확인하는 작업입니다.
the book에서 작성하라 제안한 metadata들은 **저자, 라이센스, crate 설명, 홈페이지 링크, 상세설명 링크, 저장소 링크, Readme**입니다. 
이외에도 keyword, category 등을 추가할 수 있는데, 띄어쓰기를 쓰지 않고 카테고리는 [category slugs](https://crates.io/category_slugs)에서 찾아 적으면 됩니다.
저는 저자 정보나 라이센스, 저장소 정도는 추가한 상태로, 남은 내용은 차차 업데이트하는 것으로 하고 우선 다음 단계로 넘어갔습니다.

### Packaging

다음 단계는 현재 crate가 잘 compile 되는지, upload 해도 괜찮은지 테스트하는 작업입니다
아래의 두 가지 명령어는 동일한 역할을 하는데, 이를 수행했을 때 아무런 문제가 없어야 비로소 crate가 upload될 준비가 되었다 할 수 있습니다. 
~~~ bash 
$ cargo publish --dry-run
$ cargo package
~~~
rust_rts crate는 내부에 procedural macro를 선언하는 sub-package가 존재하기 때문에, 이를 먼저 publish하고 그 version에 맞춰 rust_rts의 Cargo.toml를 작성해주어야 crate가 다른 사람들에게도 잘 작동할 수 있습니다.
위의 test를 마치고, git에 최신정보까지 commit 되어있다면 `cargo publish` 명령어를 통해 crate를 공개할 수 있습니다.

## References
