---
title: Service 
---

## 介绍

>在集群中，扫描到的 符合条件的 Service 方法就是服务。服务就是一个满足特定条件的普通方法

> 业务模块中，带有 Service 注解的类中，任意一个满足 服务条件的方法，都会生成一个对应的 LocalServiceImpl 。

__在系统中，所有的请求，最终都会被 服务接收并 处理。__

* LocalService 本地服务。
	- LocalServiceImpl 是服务的真正方法。
	- 容器中的 LocalServiceManager 管理当前容器中的所有本地服务。
* 2、RemoteService 远程服务。
	- 当进行RPC 调用，代码执行到 路由节点，判断 当前节点 属于 远程调用时， 会创建 RemoteService 来进行发送远程调用。

> 只要是 有 一个 RemoteService 发起远程调用，就肯定会有 一个 LocalService 对应处理请求。

***

## LocalService 概念

* 整个集群中，最终处理 请求的，只有 LocalService .
* 容器中的 业务模块在启动时 ，会 根据 服务注册路由，同时，所有的 服务 对应的方法，都会生成 LocalServiceImpl
* LocalServiceImpl 中 保存 当前服务 所对应的Method 和 Object 对象，当前服务被执行时，运行 Method 的 invoke 方法。
* LocalServiceManager管理 当前容器（OSGI）的所有方法（服务）

__当前容器中 如果 运行 某个业务模块，那么，当前 业务模块中 的 每一个服务，肯定会对应 一个 LocalServiceImpl . __

***

## RemoteService 概念

RemoteService 只有在 路由节点判断为 远程调用时，才会使用。

__任意一次远程 请求，无论 是否 需要等待返回值，都会创建 RemoteService。如果 需要等待返回值，则会 保存 当前 RemoteService 和 Caller 等待 接收 被调用者 返回的数据之后 再 执行 Caller .__

RemoteService 的 默认发送方法，实际上执行操作如下：

- 创建 RemoteRequest 远程请求。
- 判断 当前 请求是否需要 返回值。
	* 需要返回值：将 RemoteRequest 放到  RemoteRequestManager 中进行管理
	* 不需要返回值 ：继续执行。
- 执行 RemoteRequestImpl 方法，获取 系统默认 Channel 发送消息 给 远端。


