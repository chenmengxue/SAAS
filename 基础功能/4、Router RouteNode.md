---
title: Router RouteNode
---

## 介绍

### Router

__任意一个普通服务 就是一个 Router， 可以提供当前服务的所有节点 构成了RouteNode__

> 路由文件 保存在所有容器 的 RouterManager 中。

路由 有 多种类型：
* RouteNodeByOne 获取第一个节点。
* RouteNodeByRandom 随机获取一个节点。
* RouteNodeByWeight 按照权重获取节点 。
* RouteNodeByLocal 优先执行本地服务。

_Router 的 send(Message msg)方法，是 根据当前路由的类型，获取路由节点之后，执行路由节点的发送服务方法_

而 路由类型的设置，是在 本地方法注解上声明。设置路由类型的注解 如下：
``` java
@OtherCondition({"routeType.local"})//路由类型为 优先执行本地服务。
public void transferStationMenu(Message msg, Caller caller) {
	
}
```
>同一个服务，设置不同的路由类型，被注册多次，则，路由类型按照最后一次 扫描的生效。

***


### RouteNode

> 任意一个RouteNode，都一定对应一个 LocalService，也就是 一个 当前服务的 可执行代码。

__RouteNode属性：__

* serviceFullName 当前请求的服务名称。

* nodeContainer 当前节点所在的容器。

* nodeTag 当前节点所对应的 tag （如果当前服务or 当前 服务所在的模块模块    属于 特殊 服务or 模块，则 nodeTag 和 容器名称不同，否则，默认 路由节点的 Tag 和 容器名称相同）。

* nodeModule 当前服务 对应的模块名称

* routeCondition 当前 路由节点的 路由条件。（路由条件是 大范围，比如，当前 域（就是公司） 是否有权限 请求 当前 模块 的 所有路由。

* authority 当前节点的权限条件。（权限条件是 小 范围，当 路由条件满足时，会 判断当前用户是否具有权限请求该节点）

* localTag 当前容器tag 

* localContainer 当前容器的 tag 。

* key 当前节点对应的 模块唯一的 key （某个容器，启动 某 业务模块，可能重复启动多次，但是Key 是唯一的）

* local 如果当前节点对应的服务在 当前容器，则 直接保存当前节点服务 对应的 方法，当再次请求到当前节点时，直接执行本地服务。

***


> 当前一个RPC 调用发起时，会 根据 RPC 保存的 当前服务的路由找到合适的一个节点去执行。

__RouteNode 也是 判断 执行本地服务，还是 执行远程服务的地方。只要是 当前 路由节点 的目标 容器，等于 本容器，那么执行本地服务。__

* 当 判断为 相同容器是，根据 请求名称 ，从 LocalServiceManager 中获取 真正的  LocalService 来执行代码。
* 当判断容器不同时。
	- 创建 RemoteService 。
	- 在 RemoteService 中 创建 RemoteRequest。
	- 然后 将 RemoteRequest 通过 RemoteRequestManager 管理。
	- 发送 服务给 远端 目标容器。

