---
title: Module类与子类 和 ModuleManager
---
## Module 介绍

系统中，存在模块的概念，任何一个业务系统，都属于一个模块，一个模块，就对应一个Module子类。
> 模块属性

* key : 唯一标志 ；
* localContainer : 当前模块所在容器的名称
* localTag ：当前模块的tag（如果有需要，可以进行单独设置，如果没有需要，和容器名称相同。）
* moduleName ：模块名称
* routerCondition ：当前模块路由条件 
* 模块拥有的菜单
* 模块所需要的权限
* 等等等等 很多基本属性

__模块的基本属性，就决定了模块的功能，模块的位置，以及模块所能提供服务的 路由节点位置。__


> 所有业务模块必须执行的方法

* 安装
* 启动
* 卸载
* 停止

在此用了 Reactor事件模式，在模块子类中，根据本模块的需要，分别注册读取配置文件，扫描服务，扫描controller，扫描文件，注册服务，等等 各种事件。

__模块的启动安装卸载停止，实际上就是当前模块所注册的各种事件的启动安装卸载停止。 当然，在 事件中，并非必须全部实现 这四个方法，只实现需要实现的即可。__

> 目前的 Reactor 事件有。
* ConfigureReactor 操作配置文件
* DocumentReactor 操作文档
* RegistReactor 注册
* ScannerClassReactor 扫描类
* ScannerFileReactor 扫描普通文件
* TableReactor 注册表映射

***

## ModuleManager

__ModuleManager 的职责，是负责管理 所有 注册到 module 模块的 module 子类__
> 任意一个module 子类，都对应 一个业务模块。

ModuleManager除了负责 模块的 安装启动停止卸载 之外，还 负责

* 模块是否已经准备就绪（当前模块启动所需的前置模块是否启动）
* 获取模块的菜单和权限
* 判断模块是否自动安装，和 自动安装，自动启动。
* 操作模块中的配置文件。


















