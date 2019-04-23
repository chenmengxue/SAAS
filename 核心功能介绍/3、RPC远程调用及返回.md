---
title: RPC调用整个过程。
---

__RPC调用的整个过程如下：__

* 发起调用
* 远端（或本地）接收请求。
* 远端（或本地）执行 服务 （对于 远程的容器来说，还是 执行 容器自己的 LocalService）
* 远端发送 服务执行结果（如果执行本地服务，则没有当前步骤）
* 本地接受远端 的返回结果并处理。 

> 判断 当次RPC调用属于远程还是本地 的操作，是在 RouteNode 上进行的。

***

## 调用发起

__RPC 发起调用时，会执行按顺序执行以下操作__

* 判断 RPC调用类型，是否属于特殊调用。
	- 如果属于特殊调用，执行特殊 发送管理，根据调用类型选择不同的发送方式 
	- 如果不属于特殊调用，继续往下执行。
* 判断当前请求执行的服务，本地是否有最新路由文件。
	- 如果没有最新路由文件，则等待，并 向中心服务器 发起 特殊 RPC 调用，从 中心服务器获取路由文件。
	- 如果 本地路由文件 已经是最新，则 继续执行。
* 获取当前服务最新的路由。
* 执行 __挂载代码 1__。
	- （此处的代码挂载点随时可以挂载代码执行）当前代码挂载点的作用 有
		* 修改消息主体内容。
		* 如果当前请求节点不可用，可以进行 请求导向，改为调用可用的请求。
* 根据消息主体 和 路由类型，获取 一个可用进行RPC调用的节点。
* 判断 当前 用户 是否有 权限 请求当前节点。
	- 无权限 ，报错并返回false
	- 有权限，则 执行 RouteNode 的 send 方法。


>此系统中的 RPC 的设计使用亮点，就是 充分利用 OSGI的 特性，判断 当 请求调用方 和 请求发起方 属于同一容器时，直接执行本地代码。



***



__当判断 请求发起方 和 请求 执行方不属于相同容器，则会执行以下操作。__

* 创建RemoteService.
* 创建RemoteService 创建 RemoteRequest。
	- 如果当前请求需要 返回值，则 将 RemoteRequest 通过 RemoteRequestManager 管理。保存 当前 请求的 Caller 和 当次请求的 ID。
	- 如果当前请求不需要返回值，则 继续执行。
* 执行 创建RemoteService 的 sendRequest 方法（ 调用容器默认Channel 发送消息给目标容器） 



***



## 本地服务(在RouteNode 中如何判断属于同一容器，请查看其他文档)

__当在RouteNode 中判断，提供服务的容器和请求发起的容器属于同一容器时，会执行以下操作__

* 根据 当前请求的 服务名称，从本地 LocalServiceManager中 获取LocalService(impl) .
* 然后执行 LocalService 的rpcCall 方法，传入参数 Message 和 Caller。
* 根据条件（是否需要返回值）判断
	- 执行 Caller 的 回调方法
	- 方法执行完毕 就是整个 RPC的过程 的结束。

> 在普通的 RPC请求中，执行本容器服务 比 执行远程服务方便很多。但是特殊请求就比较复杂了。



***



## 远端接收请求 + 远端容器的本地方法执行 + 发送返回值 给 请求发起方

> 远端接受数据的是 容器默认创建的 Channel，处理请求的，是 在 容器 ServerCore启动类中，挂载到 Channel 上的不同 处理方法。

__当被调用者容器接收到请求，会做出如下操作：__

* 消费者接收到 请求，执行 Channel onMessage 方法。
* Channel 获取消息类型，找出当前请求类型 对应的 Receiver（Receiver是在容器初始化时，挂载到 Channel上的）
* 所有的 RPC请求的发起，Message 的 msgType 都是 request。对应 处理当前 消息类型的 Receiver 是 RequestReceiver，因此，执行 RequestReceiver 的 接受消息处理代码。

> 任何容器接收到任何数据都会执行前面两步。

* RequestReceiver 将 String 类型的消息转换为 Message。
* 创建 Helper （这个不过多解释，有 专门解释 Helper 的文章）
* 创建 RemoteCaller 。（此处的 RemoteCaller 是 需要传递到 服务中的参数，并且，真正给 请求发起者 发送返回消息的，正是 RequestReceiver）
* 从 Message 中 获取方法名称。
* 从 LocalServiceManager 中 获取 需要执行 当前请求的 方法。
* 执行 __挂载代码 2__ （此处是 第二处挂载代码，作用等同于 挂载代码 1）
* LocalService 执行 代码，处理请求。

> 在 LocalService 中，会 执行 Caller 的 receiveResultMessage 方法，而 此处 LocalService 的 Caller 参数，是 RemoteCaller ，在 RemoteCaller 的 receiveResultMessage ，就是 给 请求发起者返回数据。

> 此处 LocalService 执行代码是在 执行时间可控 的线程池中。当 本地服务执行时间过长，超过预设的 执行时间，则会直接报错退出。并释放线程。

* 如果本地服务执行报错，则 直接 执行 RemoteCaller 的 receiveResultMessage ，发送 错误信息日志 给 请求发起方。



***



## 调用发起方接收 被调用方的返回值。

> 消息接收的处理方式，与 请求被调用方 类似，只是 msgType不同，所以 Receiver 不同，此处的 Receiver 是 ResponseReceiver。

* String 类型消息转换成 Message。
* 获取消息的ID
* RemoteRequestManager 根据 消息的 ID 获取 缓存 的 RemoteRequest，之后，执行 对应 Caller 的 receiveResultMessage方法。






