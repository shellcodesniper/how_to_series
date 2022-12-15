---
title: 워크스페이스를 이용한 개발(점점 커지는 프로젝트) 팁1
category: rust
sub_category: workspace
order: 1
id: 7
# permalink: /Rust/workspace
---

## Rust

러스트가 1.0 으로 바뀌고 난 이후, 푹 빠진지도 이미 오래고, 해당 언어를 주력 언어로 시작 하게 된 이후로
어떤식으로 개발해야 더 완성도 높고 헷갈리지 않으며, 잘 개발될까에 대한 진짜매우많은 시도를하며 존나 많은 실패와 삽질을 한것같다

그냥 그런 경험들을 끄적거리는 내용중 하나이다.

개발자마다 개성이 있듯이 글쓴이는, main 이라는 이름과 hacking 이라는 분야를 매우 사랑해서, 프로그램 진입점과 디렉토리 이름을 주로 항상 main으로 지어둔다

러스트로 개발구조를 잡을때 이제는 이런 구조로 잡고 들어간다.

### workspace?
- 여러개의 모듈 혹은 패키지들을 묶어서 하나의 프로젝트로 구성해서 개발할 수 있는? 자세한 내용은 [CARGO WORKSPACE][https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html] 참조

### 프로젝트 트리

> main 이라는 프로그램을 출력하는 프로그램의 경우.
> # 이후는 설명이다.
>

- PROJECT_NAME/ # [ 프로젝트 루트 (WORKSPACE)]
  - Cargo.toml
  # 프로젝트를 구성하는 패키지들을 컴파일러에게 알려주고, 해당 패키지들의 공통 되는 설정을 진행
  - main/
    # main 이라는 출력물을 만들어줄 진입점을 가진 binary package 모듈이다
    - Cargo.toml
    # main 패키지의 Cargo.toml 에는 utils 를 사용하기위한 설정과 의존성 설정 등을 담고있을것
    - src/
    # main 패키지의 src 파일트리
      - main.rs
      # main 패키지의 main 소스이며 entrypoint 인 main 함수를 들고있는 파일 (진짜 공대 개그 하려는거 아니고, main이 좋아서 그런거)
      - interfaces/
      # 메인 패키지 내에서 사용되는 인터페이스를 모아둔 internal 패키지가 되겠다
        - mod.rs
        # interfaces를 use 할수있도록 ~ 머머
  - utils/
  # main 에서 쓸수 있는 PROJECT_NAME 프로젝트 의 또다른 패키지다 이는 라이브러리로 할생각
    - src/
      - lib.rs
      # library 패키지는 lib.rs 파일을 참고하게되므로, lib.rs 를 놔두자
      - single_package.rs
      # use 함수로써, 사용하고싶지만 파일트리로 다시 디렉토리 안에 넣고싶지 않다면 이런식으로 하는건 어떨까
      - multiple/
      # 만약 패키지를 폴더로 만들어서 연결되는게 많다면, multiple.rs 를 만들지말고, multiple/mod.rs 를 만들어서 사용하자.
        - mod.rs
        # multiple 이라는 패키지 모듈의 내용을 사용하기 위한 내용들을 담고있음.

    
