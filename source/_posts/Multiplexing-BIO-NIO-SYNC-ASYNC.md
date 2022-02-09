---
title: Multiplexing-BIO-NIO-SYNC-ASYNC
date: 2022-02-08 20:39:09
tags: Network
---

# 多路复用、BIO、NIO、同步和异步

最近发现这些概念很容易混淆，想自己好好梳理一遍。

下面以Java BIO/NIO API的视角来理解多路复用、BIO、NIO、同步和异步。

## 多路复用

多路复用技术出现前，连接的读操作是与线程绑定的，假设有N个连接需要执行read()，那么需要N个线程来完成。

多路复用技术出现后，连接的读操作和可读状态可以分离开了，我们可以使用一个线程来操作多路复用器（Selector）管理N个连接的可读状态。但是对于每一个可读状态的连接，使用BIO的情况下，（为了不阻塞Selector线程）我们仍需要N个线程来完成读写操作。如果使用NIO，读操作只会读取可读的数据并将数据存到Buffer，因此每一个连接的读操作都不会阻塞Selector线程。对于Buffer的管理，我们只需要把满足业务条件（达到指定长度）Buffer的数据交给某个线程池处理即可（事实上这就是Netty EventGroup的实现原理）。

> 注意：目前讨论的都是同步编码方式

这样一来，所需的线程数量就可以减少、上下文切换随即减少、系统吞吐量提高、内存使用率降低。

### 多路复用 syscall

- select
- poll
- epoll (Linux) / kqueue (FreeBSD)



## BIO vs. NIO

BIO和NIO的概念与线程无关。

### BIO

传统的读写操作都是BIO，读写操作会阻塞线程直到读写完成。

Java API中的BIO是面向Stream的，读写对应InputStream和OutputStream。

### NIO

NIO关注可读可写状态

非阻塞的读写操作，即只读取可读取的部分和只写可写的部分，如果没有数据可读写，立即返回。

Java API中的NIO是面向ByteBuffer的，读写操作都只是复制Buffer，剩下的交给kernel处理，所以不会阻塞。

## 同步 vs. 异步

同步和异步的概念与线程有关。

### 同步Sync

普通的代码，从上往下执行，就是同步的。

### 异步Async

异步的代码是无序的，通常和事件event、回调callback绑定。

当出现异步时，其背后一定有一个线程来完成异步操作。

## Netty

Netty是Java的网络编程框架，它优雅地实现了多路复用、NIO和异步，开箱即用。

## 总结

多路复用、NIO和异步这几个词经常在一起出现，比如Netty的实现，通过减少线程的方式增加吞吐量和减少内存使用。

## Reference

- [What is the exact use of java nio package when already methods are available with io package](https://stackoverflow.com/questions/10372066/what-is-the-exact-use-of-java-nio-package-when-already-methods-are-available-wit)
- [The C10K problem](http://www.kegel.com/c10k.html)
- [Java NIO vs. BIO](http://tutorials.jenkov.com/java-nio/nio-vs-io.html)
- [多路复用、非阻塞、线程与协程](https://www.nosuchfield.com/2019/01/09/Multiplex-and-non-blocking-and-threading-and-coroutine/)
