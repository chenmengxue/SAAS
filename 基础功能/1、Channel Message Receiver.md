---
title: Channel 和 Message 和 Receiver
---

> 集群中，真正进行数据发送接收的 是 Channel，而 在 Channel 上发送的 数据体是 Message，针对不同的Message ，所不同的 处理机制，就是 Receiver

## Message 介绍

__Message是消息主体。系统中，所有的RPC 调用所用来交互的 真正数据只有Message。__

> Message 属性
* ID 消息id ，消息分为主动请求，和 接受被请求者的返回消息。
* methodFullName 当前消息请求到远程需要执行的方法
* routerName 路由名称（特殊用处，当前消息如果是发送给中心服务器，需要 指定当前参数）
* msgType 消息类型，不同的消息类型，对应不同的处理机制，而处理机制，是 挂载 到 Channel上的。
* sourceTag 请求来源地址
* targetTag 请求目标地址
* msgString 真正的消息实体
* attributeMap 消息实体补充参数
* states 消息状态
* desc 与状态匹配的描述
* sendType 消息发送类型（默认 就是 RPC请求，或者，发给中心服务器，发给 容器服务）
* …… 还有一些其他的参数

_消息的 ID ，可以在 接受 被请求者返回的消息时，找到 本地保存的 请求返回后对应的处理方法。
消息的其他基本属性，就 确定了 消息的来源，和 目标地址，以及，在目标地址要执行的操作，和 对应操作获取之后应该执行的方法 等等 信息，方便消息 进一步的操作。_

***

> Message 方法
* 构造方法（String jsonString) ，在 消费者接收到String 格式的 数据之后，可以转换成 Message 消息。
* 重写toString() 消息转换成 String 方便 Channel 发送消息

_消息的方法，主要是方便 Message 与 String 格式 的 数据进行转换，因为，真正的发送底层是RocketMQ，发送的数据格式就是 String。_

***

## Channel 介绍

>  Channel 的初始化时，会将 channel 与 RocketMQ 的生产者和消费者绑定。

_Channel 底层是用 RocketMQ 实现的。_

__这里需要注意：__  

* 任意 一 Channel绑定 的 生产者和消费者 属于同一个Topic。
* 一个 Channel只能监听一个 tag 。
* 在进行远程调用的实现中，默认的 tag ，都是指容器名称。
* 特殊模块的tag可以特殊设置，但是，特殊设置的 tag 不能与 所有的 容器名称 和 其他的tag重复。



> 系统默认的Channel 初始化操作，在 Server Core 模块的启动类中进行。

* 若当前模块需要使用特殊 Channel ，则在当前模块启动类中进行初始化。
* 任何容器 均会初始化默认Channel，RPC 消息发送由默认 Channel执行。
* 在系统中，任意 远端RPC调用，都是调用 Channel.sendMsg（Message msg) 方法，发送的消息主体必须是 Message。

__Channel接收到的 Message 的 msgType 不同，对应的处理方法就不同，就会有不同的 Receiver 来处理不同的消息  __

>Receiver的注册操作，是在 ServerCore 项目的启动类中实现的。

***

## Receiver 介绍

> Receiver 只有一个 onMessage 的方法，每种 Receiver代表不同的处理方式，用来 处理不同类型的 数据。

__系统中主要的 Receiver 有如下几种：__

* ContainerReceiver  对 当前容器 进行某些操作。
* ContainerReturnReceiver  接受 远端操作容器之后的返回值。
* LoadDomainReceiver 加载域（给当前容器赋予某一个公司）
* MappingReceiver 给当前容器设置 数据映射。
* MappingReturnReceiver  等待 远端映射设置  的返回。
* ModuleReceiver 操作容器模块。
* ModuleReturnReceiver 等待远端操作模块的返回。
* RemoveUsercacheReceiver 删除本容器的 用户数据缓存。
* RequestReceiver 接收请求。
* ResponseReceiver 接收  远端服务的返回。
* RouteReceiver 更改本地路由。

> 跟据名称可以很明显的 知道 ，当前 Receiver 的处理操作是什么。



over.
