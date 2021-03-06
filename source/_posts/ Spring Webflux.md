---
title: Spring Webflux
date: 2021-10-05 16:38:43
tags:
- spring
- java
categories:  SpringWebflux
---

**1.响应式编程是什么？**

- 响应式编程（**reactive programming**）是一种基于数据流和变化传递的声明式的编程范式

  本来数据是我们自行处理的，后来我们把要处理的数据抽象出来（变成了数据流），然后通过**API**去处理数据流中的数据（是声明式的,如：

  ```java
    int sum2 = IntStream.of(nums).parallel().sum();
  ```

​      将数组中的数据变成数据流，通过显式声明调用**.sum()**来处理数据流中的数据，得到最终的结果。

- 响应式编程是关于非阻塞应用程序的，这些应用程序是异步的、事件驱动的，并且需要少量的线程来垂直伸缩(即在 JVM 中)，而不是水平伸缩(即通过集群)。

  > 工作太多，做不完时，水平伸缩就相当于加人，垂直伸缩相当于加班。

### 2.响应式流（Reactive Streams） 是什么?

<!-- more -->

- 响应式流是JDK9引入的，基于发布-订阅者模式的一套数据处理的机制。

- 响应式流从2013年开始，作为提供非阻塞背压的异步流处理标准的倡议。 它旨在解决处理元素流的问题——如何将元素流从发布者传递到订阅者，而不需要发布者阻塞，或订阅者有无限制的缓冲区或丢弃。

> 背压：说白了就是一种反馈，发布者和订阅者之间的一种互动

> 之前的老模式，订阅者很被动，发布者给订阅者多少他就消费多少，不能多也不能多少。
>
> Reactive Streams就可以做到发布者和订阅者之间可以进行交流，订阅者可以告诉发布者我需要多少数据，订阅者处理完了，可以再向发布者要，没处理完就不要给我。
>
> 起到了一种调节流量的作用，不会导致发布者数据太多，订阅者处理不完浪费，或者直接把订阅者压垮的场景。

### 2. Spring Webflux 是什么？

Spring WebFlux是[Spring MVC](https://rumenz.com/java-topic/spring-mvc-tutorial/index.html)并行版本，并支持完全无阻塞的反应流的web框架。 它支持背压概念，可以处理大量的并发连接，并使用**[Netty](https://netty.io/)**作为内置服务器来运行响应式应用程序。 

### 3.Spring Webflux 和Spring mvc 的关系

<img src="https://spring.io/images/diagram-reactive-1290533f3f01ec9c57baf2cc9ea9fa2f.svg" style="zoom: 25%;" />

Spring MVC

- 构建于 Servlet API 之上
- 同步阻塞 I/O 模型, 认为应用会阻塞当前线程，所以一个 Request 对应一个 Thread，需要有一个含有大量线程的线程池

Spring WebFlux

- 构建于 Reactive Streams Adapters 之上
- 异步非阻塞 I/O 模型，认为应用不会阻塞当前线程，所以只是需要一个包含少数固定线程数的线程池 (event loop workers) 来处理请求
