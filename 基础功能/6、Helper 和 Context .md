---
title: Helper 和 Context
---

## Helper 

__Helper 是 platform 项目中，比较核心的功能之一，主要负责功能如下：__

* 保存当前线程的Context。
* 发送 远程RPC调用。
* 提供 本次请求的 HttpServletRequest，HttpServletResponse。
* 提供 页面请求的参数 Parameters
* 提供 当前 请求者的 用户，域（公司），信息。

系统第一次创建 Helper是在 BaseServlet中，

## Context 