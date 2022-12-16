---
title: Rust Advanced - Async Trait 에 관하여
category: rust
sub_category: advanced
order: 1
id: 9
# permalink: /Rust/workspace
---

## Rust

stable 채널에서 (현재 1.65 가 stable) 구현할때

tokio에서는 어떻게 구현하나 를 참고해보면서 따라가며 살펴본 내용들.





## Rust 에서의 Async

- Futures Lib의 Poll.
- Poll Status
  - _before Poll::Pending [ 임의로 적은 상태 ]
    - "wake<Fn()>" 이 실행되기 이전
  - Poll::Ready [ 준비된 상태, 뭔가 배출 가능 ]
    - tick 에 따라, 작업을 진행하면서, 결과물을 yield 하는 상태

























#### References

- [async-book][https://rust-lang.github.io/async-book/04_pinning/01_chapter.html]
