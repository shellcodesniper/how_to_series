---
title: Rust Advanced - Async Trait 에 관하여
category: rust
sub_category: advanced
order: 1
id: 9
# permalink: /Rust/workspace
---

## Rust

TOKIO-RS 의 AsyncRead 부분을 구현하는 쪽을 이용하여 STABLE 채널 [ 현재 1.65 ] 에서

async trait 을 구현해보도록 한다.

`소스코드 출처 : https://github.com/tokio-rs/tokio`

이하 소스코드는 바로 빌드 & 테스트 가능하도록 짜집기 해두었다.

> 
>
> 의존성으로, `pin-project-lite` 를 필요로하다.
>
> 







`src/lib.rs`

```rust
use std::{
  io::{ self, IoSlice },
  ops::DerefMut,
  pin::Pin,
  task::{ Context, Poll },
  future::Future,
  marker::{ PhantomPinned, Unpin },
};
use pin_project_lite::pin_project;
use tokio::io::ReadBuf;



// ? : MACRO Poll:Ready(t) => t
  macro_rules! ready {
  ($e:expr $(,)?) => {
    match $e {
      std::task::Poll::Ready(t) => t,
      std::task::Poll::Pending => return std::task::Poll::Pending,
    }
  };
}



// ? : 이친구가 read.await 이런식으로 쓰일 애인가봄
// LIFETIME 'a
// R : AsyncRead + Unpin + ?Sized
// Return : Read<'a, R>
pub(crate) fn read<'a, R>(reader: &'a mut R, buf: &'a mut [u8]) -> Read<'a, R>
where
  R: AsyncRead + Unpin + ?Sized,
{
  Read {
    reader,
    buf,
    _pin: PhantomPinned,
  }
}



// TODO : 여기 봐줘야됨~
pin_project! {
  #[derive(Debug)]
  #[must_use = "futures do nothing unless you `.await` or poll them"]
  pub struct Read<'a, R: ?Sized> {
    reader: &'a mut R,
    buf: &'a mut [u8],
    #[pin]
    _pin: PhantomPinned,
  }
}




impl<R> Future for Read<'_, R>
where
  R: AsyncRead + Unpin + ?Sized,
{
  type Output = io::Result<usize>;

  fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<io::Result<usize>> {
    let me = self.project();
    let mut buf = ReadBuf::new(me.buf);
    ready!(Pin::new(me.reader).poll_read(cx, &mut buf))?;
    Poll::Ready(Ok(buf.filled().len()))
  }
}




pub trait AsyncRead {
  fn poll_read(
    self: Pin<&mut Self>,
    cx: &mut Context<'_>,
    buf: &mut ReadBuf<'_>,
  ) -> Poll<io::Result<()>>;
}




macro_rules! deref_async_read {
  () => {
    fn poll_read(
      mut self: Pin<&mut Self>,
      cx: &mut Context<'_>,
      buf: &mut ReadBuf<'_>,
    ) -> Poll<io::Result<()>> {
      Pin::new(&mut **self).poll_read(cx, buf)
    }
  };
}




impl<T: ?Sized + AsyncRead + Unpin> AsyncRead for Box<T>
{
  deref_async_read!();
}




impl<T: ?Sized + AsyncRead + Unpin> AsyncRead for &mut T
{
  deref_async_read!();
}




impl<P> AsyncRead for Pin<P>
where
  P: DerefMut + Unpin,
  P::Target: AsyncRead,
{
  fn poll_read(
    self: Pin<&mut Self>,
    cx: &mut Context<'_>,
    buf: &mut ReadBuf<'_>,
  ) -> Poll<io::Result<()>> {
    self.get_mut().as_mut().poll_read(cx, buf)
  }
}




impl AsyncRead for &[u8]
{
  fn poll_read(
    mut self: Pin<&mut Self>,
    _cx: &mut Context<'_>,
    buf: &mut ReadBuf<'_>,
  ) -> Poll<io::Result<()>> {
    let amt = std::cmp::min(self.len(), buf.remaining());
    let (a, b) = self.split_at(amt);
    buf.put_slice(a);
    *self = b;
    Poll::Ready(Ok(()))
  }
}




impl<T: AsRef<[u8]> + Unpin> AsyncRead for io::Cursor<T>
{
  fn poll_read(
    mut self: Pin<&mut Self>,
    _cx: &mut Context<'_>,
    buf: &mut ReadBuf<'_>,
  ) -> Poll<io::Result<()>> {
    let pos = self.position();
    let slice: &[u8] = (*self).get_ref().as_ref();

    if pos > slice.len() as u64 {
      return Poll::Ready(Ok(()));
    }

    let start = pos as usize;
    let amt = std::cmp::min(slice.len() - start, buf.remaining());
    let end = start + amt;
    buf.put_slice(&slice[start..end]);
    self.set_position(end as u64);

    Poll::Ready(Ok(()))
  }
}




#[cfg(test)]
mod tests {
  #[tokio::test]
  async fn test_fn() {
    println!("ASDFASDFASDF");
    let mut buf = [0; 1024];
    let mut reader = std::io::Cursor::new(&b"hello world"[..]);
    let n = super::read(&mut reader, &mut buf).await.unwrap();
    assert_eq!(&buf[..n], b"hello world");
  }
}





```





## ㅂ..분석

결국 read 라는 함수 하나를 만들기 위해 이런 짓을 해야된다는것,



```rust
use std::{
  io,
  future::Future,
  pin::Pin,
  marker::{ PhantomPinned, Unpin },
  ops::DerefMut,
  task::{ Context, Poll },
};
use pin_project_lite::pin_project;
use tokio::io::ReadBuf;
use util::ready;

// NOTE : ASYNC 구현체 시작

// TYPE : async 함수 본체
pub(crate) fn read<'a, R>(reader: &'a mut R, buf: &'a mut [u8]) -> MyAsync<'a, R>
where
  R: AsyncTrait + Unpin + ?Sized,
{
  MyAsync {
    reader,
    buf,
    _pin: PhantomPinned,
  }
}

// TYPE : async trait 을 impl 당한 구조체
pin_project! {
  #[derive(Debug)]
  #[must_use = "futures do nothing unless you `.await` or poll them"]
  pub struct MyAsync<'a, R: ?Sized> {
    // TYPE : .project() 를 통해 이쪽으로 접근 가능
    reader: &'a mut R,
    buf: &'a mut [u8],
    #[pin]
    _pin: PhantomPinned,
  }
}

// TYPE : async function을 가지고있는 trait
pub trait AsyncTrait {
  fn poll_read(
    self: Pin<&mut Self>,
    cx: &mut Context<'_>,
    buf: &mut ReadBuf<'_>,
  ) -> Poll<io::Result<()>>;
}

// NOTE : 여기까지가 ASYNC 구현체
// ======================================================================================================


// TYPE : 여기는 .await 를 통해 호출되는 함수를 구현하는 곳인듯함
impl<R> Future for MyAsync<'_, R>
where
  R: AsyncTrait + Unpin + ?Sized,
{
  type Output = io::Result<usize>;

  fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<io::Result<usize>> {
    let me = self.project();
    let mut buf = ReadBuf::new(me.buf);
    ready!(Pin::new(me.reader).poll_read(cx, &mut buf))?;
    Poll::Ready(Ok(buf.filled().len()))
  }
}


// TYPE : 여기는 매크로를 이용해, async_trait 을 구현하는 곳인듯함
macro_rules! deref_async_read {
  () => {
    fn poll_read(
      mut self: Pin<&mut Self>,
      cx: &mut Context<'_>,
      buf: &mut ReadBuf<'_>,
    ) -> Poll<io::Result<()>> {
      Pin::new(&mut **self).poll_read(cx, buf)
    }
  };
}

// TYPE : 여기는 Box<T> 타입을 다룰때에 대해 ?
impl<T: ?Sized + AsyncTrait + Unpin> AsyncTrait for Box<T>
{
  deref_async_read!();
}

// TYPE : 여기는.. &mut T 타입을 다룰때에 ?
impl<T: ?Sized + AsyncTrait + Unpin> AsyncTrait for &mut T
{
  deref_async_read!();
}

// TYPE : 여기는 Pin<P> 타입을 다룰때에 ?
impl<P> AsyncTrait for Pin<P>
where
  P: DerefMut + Unpin,
  P::Target: AsyncTrait,
{
  fn poll_read(
    self: Pin<&mut Self>,
    cx: &mut Context<'_>,
    buf: &mut ReadBuf<'_>,
  ) -> Poll<io::Result<()>> {
    self.get_mut().as_mut().poll_read(cx, buf)
  }
}


// TYPE : 여기는 반환 타입이 &[u8] 이고 Cursor<T> 의 input 일때에 대한 impl
impl<T: AsRef<[u8]> + Unpin> AsyncTrait for io::Cursor<T>
{
  fn poll_read(
    mut self: Pin<&mut Self>,
    _cx: &mut Context<'_>,
    buf: &mut ReadBuf<'_>,
  ) -> Poll<io::Result<()>> {
    let pos = self.position();
    let slice: &[u8] = (*self).get_ref().as_ref();

    if pos > slice.len() as u64 {
      return Poll::Ready(Ok(()));
    }

    let start = pos as usize;
    let amt = std::cmp::min(slice.len() - start, buf.remaining());
    let end = start + amt;
    buf.put_slice(&slice[start..end]);
    self.set_position(end as u64);

    Poll::Ready(Ok(()))
  }
}
// NOTE : 아마 여기서 사용할 수 있게 해주는것같은데...?





#[cfg(test)]
mod tests {
  #[tokio::test]
  async fn test_fn() {
    println!("ASDFASDFASDF");
    let mut buf = [0; 1024];
    let mut reader = std::io::Cursor::new(&b"hello world"[..]);
    let n = super::read(&mut reader, &mut buf).await.unwrap();
    assert_eq!(&buf[..n], b"hello world");
  }
}





```



















































