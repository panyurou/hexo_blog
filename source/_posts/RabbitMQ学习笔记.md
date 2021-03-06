---
title: RabbitMQ学习笔记
date: 2021-10-27 21:35:27
tags: 中间件
categories: rabbitMQ
---

# **什么是RabbitMQ？** 

RabbitMQ是一款开源的，Erlang编写的，基于AMQP协议的消息中间件

# **rabbitmq 的使用场景** 

> - 服务间异步通信 
>
> - 顺序消费
> - 定时任务 
> - 请求削峰 

<!--more -->

# rabbitmq 的优点

* 异步解偶。 相比于传统的串行、并行执行，可以提高吞吐量
* 流量削峰。可以通过消息队列长度控制请求量；可以缓解短时间内的高并发请 求。
* 应用解偶。系统间通过消息通信，不用关心其他系统的处理。 
* 消息通讯 - 消息队列一般都内置了高效的通信机制，因此也可以用在纯的消息通讯。比如实现点对点消息队列，或者聊天室等。 

> 主要是：解耦、异步、削峰。 

# 为什么选择rabbitmq？

* 支撑高并发、高吞吐、性能很高，同时有非常完善便捷的后台管理界面可以使用。 
* 支持集群化、高可用部署架构、消息高可靠支持，功能较为完善。
* 开 源的，比较稳定的支持，活跃度也高

> 缺点： 确实 erlang 语言阻止了大量的 Java 工程师 去深入研究和掌控它，对公司而言，几乎处于不可控的状态，

# RabbitMQ的基本概念

* Broker： 简单来说就是消息队列服务器实体 

* Exchange： 消息交换机，它指定消息按什么规则，路由到哪个队列 

* Queue： 消息队列载体，每个消息都会被投入到一个或多个队列 

* Binding： 绑定，它的作用就是把exchange和queue按照路由规则绑定起来 

* Routing Key： 路由关键字，exchange根据这个关键字进行消息投递 
* VHost： vhost 可以理解为虚拟 broker ，即 mini-RabbitMQ server。其内部 均含有独立的 queue、exchange 和 binding 等，但最最重要的是，其拥有独立的 权限系统，可以做到 vhost 范围的用户控制。当然，从 RabbitMQ 的全局角度，类似mysql中的database创建一个table 需要指明用哪个数据库。

# channel信道

- 信道是生产者/消费者与rbabitmq通信的渠道。生产者publish或者消费者consume一个队列都是通过信道完成的。
- 多线程时，rabbitMQ在一条tcp上建立多个信道来处理多线程。
- 每个信道在rabbitmq上有唯一的id,保证一个信道对应一个线程使用。

# 交换机

- 交换机就类似是路由器，他会根据路由键（在rabbitMQ就是routing key），将消息分发到相应的队列上去。

- 交换机的四种类型
  - fanout: 把所有发送到该交换器的消息路由到所有与该交换器绑定的队列中。(1:N)
  - Direct :   把消息路由到BindingKey和RoutingKey完全匹配的队列中。(1:1)
  - topic: 可以根据模糊匹配，可以将多个消息路由到同一个队列，比如一个队列的routing_key是''.test"那么凡是到达路由器的消息的后缀为“.test”，都会进到这个队列。
    - “* ”只能是一个单词，如“”aa.test“
    - “#”可以是>= 0的单词，如“”test“,"aa.bb.test"
  - `headers`:不依赖路由键匹配规则路由消息。是根据发送消息内容中的`headers`属性进行匹配。性能差，基本用不到。

# **RabbitMQ的6种工作模式** 

**一.simple模式（即最简单的收发模式）**

一个生产者，一个消费者，通过队列收发消息。不常用

**二.work工作模式(资源的竞争)** 

* 一个生产者，多个消费者。

* 消费者1,消费者2同时监听同一 个队列,消息被消费。C1 C2共同争抢当前的消息队列内容,谁先拿到谁负责消费 消息
* (隐患：高并发情况下,默认会产生某一个消息被多个消费者共同使用,可以设置一个开关(syncronize) 保证一条消息只能被一个消费者使用)。 

**三.publish/subscribe发布订阅(共享资源)**

1、每个消费者监听自己的队列； 

2、生产者将消息发给broker，由交换机将消息转发到绑定此交换机的每个队列，每个绑定交换机的队列都将接收到消息。

**四.routing路由模式** 

根据routing_key 进行匹配，生产者，发送消息的时候会制定routing_key，交换机根据routing_key，去匹配绑定改routing_key的队列

,只能匹配上路由key对应 的消息队列,对应的消费者才能消费消息

**五.topic 主题模式(路由模式的一种)** 

.交换机根据key的规则模糊匹配到对应的队列,由队列的监听消费者接收消息消费。类似sq l的模糊匹配

**六.RPC模式(路由模式的一种)** 

实现不同服务间的远程调用

## 消息的分发策略

- 消息的分发策略

    假设队列里有100条消息，有 A,B,C   3个队列

  - 发布订阅。三个队列都收到100条
  - 轮训分发。3个队列都是至少33条，剩下一条随机，不论你数据库性能怎么样，大家接受的都是公平的
  - 公平分发。根据服务器性能，去分发，哪个性能高，哪个处理的消息可能就多，能者多劳，会造成数据倾斜
  - 重发。发送消息中出现了异常后，消息没有得到应答，就会重发，kafka不支持
  - 消息拉取。就是RPC去拉取数据

  > Rabbitmq以上集中策略都支持，且是开源的
  >
  > kafka速度最快
  
  

# RabbitMQ的高级特性

##  消息的可靠性投递

1. 作为发送方，我们希望杜绝任何消息丢失或者投递失败，因此rabbitMQ 给我们提供了两种方式，来控制消息投递的可靠性。
   * confirm确认机制
   * return 退回模式

2. rabbitMQ 的整个消息投递的路径为：`producer-> rabbitmq broker -> exchange->queue->consumer`

   * 消息从producer ->  exchange 会返回一个confirmCallBack

   * 消息从exchange -> queue 会反回一个returnCallBack

     我们将利用这两个CallBack来控制消息的可靠性投递

## Consumer Ack 消费端收到消息后的确认方式

1. 有三种确认方式
   * 自动确认 acknowledge = "none"。 消息一旦到达consumer就会被确认，并将对应的message 从消息缓存中移除，实际场景中，很可能消息被收到，但是处理业务时异常，这种确认机制下，消息就会丢失。
   * 手动确认acknowledge = "manual"。设置了手动确认，则需要在业务处理成成功后，手动调用`channer.basicAck()`,如果出现异常则调用`channer.basicNAck()`,设置消息是重新返回队列，还是直接丢掉。
   * 根据异常情况确认 acknowledge = "auto"

##  消费端限流

* 在<rabbit:listener-container> 中配置`prefetch`属性设置消费端一次拉取多少消息
* 消费端的确认模式需要是手动确认

## TTL

1. TTL全称：time to live 消息存活时间或消息过期时间
2. 消息达到了存活时间后，如果还没被消费，会被自动移除
3. RabbitMQ卡哇伊对消息设置过期时间，也可以对整个队列设置过期时间。

## 死信队列

* 当消息成为死信后，可以被重新发送到一个交换机，这个交换机就是死信交换机，它绑定的队列就是死信队列

* 成为死信的条件：
  * 消息达到了存活时间，还没有被消费。
  * 消费者拒收消息，并且不重回队列。  
  * 队列到达了指定的长度限制

## 延迟队列 

延迟队列，消息进入队列后，不会立即被消费，而是等到一定的时间，才会被消费。

使用场景：

1. 用户下单后，30分钟未支付，取消订单，回滚库存
2. 新用户注册7天后，发送短信问候

> 当然上面的场景也可以用定时器实现

rabbitmq现在不支持延迟队列，延迟队列的实现需要借助TTL和死信队列。具体实现流程：

* 用户下单，把消息发送到Queue1中，不设置Consumer1，设置Queue1队列里的消息存活时间为30分钟，等待30分钟后，消息成为死信。
* 死信的消息发送到Queue2，添加Consumer2监听Queue2

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gwld73gtjaj31ga0o440f.jpg" style="zoom:43%;" />

#### 死信队列和延时队列的区别

* 死信队列，监听的是Queue1,成为死信的消息会被丢到DLX中，或者不处理自己清理掉
* 延迟队列，监听的是死信队列

<hr/>



## 消息积压和解决

#### 为什么会有消费积压？

1. 消费者宕机了
2. 发送方发送流量太大
3. 消费者能力不足

#### 解决方式

1. 上线更多的消费者
2. 上线专门的队列消费服务
3. 先取出来放到数据库，再慢慢处理



## 消息的幂等性

造成消息重复的根本原因是：网络不可达。 

所以解决这个问题的办法就是绕过这个问题。那么问题就变成了：如果消费端收 到两条一样的消息，应该怎样处理？ 

方案一： 利用一张日志表来记录已经处理成功的消息的 ID，如 果新到的消息 ID 已经在日志表中，那么就不再处理这条消息。 

方案二：

* 第一次 执行更新语句的是一样，version =1 

  ```
  update account set price = price -100, version = version + 1, where id = 1 and version = 1
  ```

- 第二次, 执行更新语句的是一样，version 已经变成了2，此时找`where version = 1 ` 就无法找到

  ```
  update account set price = price -100, version = version + 1, where id = 1 and version = 1
  ```



## 中间件

- 中间件
  - 是一种应用于分布式系统的基础软件。
  - 常见的中间件：mysql，rabbit MQ
- 怎么选择中间件
  - 可以通信，跨平台。比方两个项目一个java，一个go之间要通信，就要遵循同一种协议
  - 高可用
    - 是否拥有持久化。比方中间件挂了，重启后是否可以把消息重新存储起来的能力
    - 支持集群。系统cpu不够用了，就得搭集群
  - 有分发能力，多个系统，往那个系统去发送消息

## 协议

- 网络协议三要素：
  - 语法：用户数据的结构与形式，如：http中规定了请求和响应报文的格式
  - 语义：规定了何种信息需要对应发出何种响应，如：请求get要把参数放在url中，post把参数放在body中
  - 时序：事件的执行顺序，如：先有请求后有响应

- 为什么消息中间件不用http？
  - http的请求和响应报文比较复杂，有cookie, 状态码，响应码这些，但消息中间件：只需要接受消息，存储消息，分发消息，不需要这么复杂
  - http大部分是短链接，不利于出现故障时消息持久化

- AMQP（advanced message. Queuing protocol） 高级消息队列协议
  - 采用Erlang，底层是C，速度很快
  - 特性
    - 支持分布式事务
    - 消息持久化
    - 高性能高可靠的消息处理优势

- kafka 协议
  - 基于TCP/IP的二进制协议，消息内部由长度分割，由基本数据类型构成
  - 特性
    - 结构简单
    - 解析速度快
    - 消息持久化
    - 不支持事务
