---
title: web
---

## 介绍


__真正提供Web 服务的 是 web4osgi 项目__

> 提供web功能，由 jetty 实现。

web 功能，在集群中，拆分为 三部分 ：

* 核心部分 ：CoreWeb 功能，提供 所有 Web 功能的基础，主要包含
	- Controller 注解
	- 扫描 业务模块中的 Controller注解 标注的类，可请求的 URL 和 当前 URL 对应的 Reactor（此处的 Reactor 是可以通过Web 请求的执行器）
	- 使用 ReactorManager 管理所有的 Reactor。
* Platform 项目部分：
	- BaseController 封装 所有Controlle 所公用的方法。
	- Context web 请求的上下文，其中封装了 Servlet 3.0 的 AsyncContext，主要负责 对页面返回数据
	- WebCaller web 的 Caller 继承自 LocalCaller
	- Helper web方面功能 的超级辅助类，负责 Controller 的 远程调用，负责维护请求的上下文，负责管理 当前 登录者 的 用户，角色，公司，权限 信息。
* web4osgi 项目：
	- 提供 web 服务
	- 提供 web 页面

***


在 业务模块的 module 子类中，设置 扫描事件时，增加 扫描Controller 事件，则业务模块初始化启动时，会 扫描 所有带有 Controller 注解的类，满足条件的方法，会生成 URL 和 对应当前 URL的 Reactor。

__web功能 的正常流程是。__

* 业务模块启动，通过 module 模块的扫描 方法，扫描 当前 业务模块的 controller ，（扫描 子类在 coreWeb 模块中）
* 在 controller 扫描是，所有符合条件的方法，会生成 Reactor 和 对应的 URL，保存到 CoreWeb 模块的 ReactorManager 中。
* 启动 web4osgi 模块。当前 容器 对外提供web 服务。
* BaseServlet 拦截所有请求，获取 URL，从 ReactorManager 中，根据请求的 URL 找到对应的 Reactor 执行方法。
* 方法 执行完成后，返回数据到页面。

当前 web 中，用的 Servlet 3.0 异步AsyncContext。