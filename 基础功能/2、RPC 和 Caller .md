---
title: RPC 和 Caller  
---

## RPC介绍

>RPC 类，就是 RPC 模块的核心


__在系统中，所有的 RPC 请求，都是 由RPC 类发起。__

* 1、手动拼装Message(手动指定 目标tag 手动设置 localTag 等等其他所有信息 )  发送给远程指定 地址 。  sendMsg(Message msg,Caller caller);
* 2、手动指定 Channel 发送消息。send(String channelName,Message msg)
* 3、指定 调用方法 和 消息 。 call(String methodFullName, Message msg)
* 4、指定 调用方法 和 消息 和 被调用方法 执行 结果返回后，请求者 接受到 返回数据的处理方法（即Caller）。 call(String methodFullName,  Message msg, Caller caller)

_1 和 2 ，都属于 特殊的消息发送类型， 需要手动设置 要发送的 Message 中的关键参数。_

_3 属于正常的 RPC 远程调用， 只需要 设置 需要调用的方法，要发送的消息主体，如果 当前调用有返回值，则 设置 接受到返回值 之后的处理方法。_

> 如果 当前 RPC 调用，需要 有返回值处理的 方法，则需要 第三个参数 Caller



## Caller
> Caller 是当 进行 RPC 调用，并且 需要远端返回值的时候，使用。

__Caller的子类有三种，分别在不同的模块，处理不同的逻辑。__

* LocalCaller ： 在写RPC远程调用时，第三个参数 使用，主要写 接收远端消息返回之后，对 消息的处理逻辑代码。LocalCaller 对应 具体的 对 Message 的操作代码。

* RemoteCaller ： 本地 接受到远端请求之后 （本地为 被调用者，远端为 发起调用者），本地 代码执行结束，给 远端发送 执行结果 时 使用。RemoteCaller 主要是 将本地 方法执行结果 发送给 请求调用者。

* WebCaller ： 当前 Caller 是 LocalCaller 的子类，用法 和 LocalCaller 类似，但是，在 WebCaller 中，会 保存 Context，等待 接收到 远程被调用者 发送消息 之后，处理完成，需要 给 浏览器页面 返回数据。Context 中，封装了 Servlet 3.0 的 AsyncContext.

__Caller 负责在 RPC请求时的权限验证。__

> Caller 会保存当前请求用户的权限信息，然后 与 当前 调用的 执行节点 所需要的权限进行比较，当 权限匹配时，才会进行 RPC 调用。



