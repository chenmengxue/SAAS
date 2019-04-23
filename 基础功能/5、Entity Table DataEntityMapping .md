---
title: Entity Table DataEntityMapping
---

## Entity 介绍

__万物皆对象 是 Java 的核心，而在 当前集群中也有一个核心，一切数据，都是 Entity。__

> Entity 中， 最主要属性 document。Document 是 Mongo 的类。Entity 对象的实际数据，是存在 document 里面的，document 是 key-value 结构。

Entity 在当前系统中，可以用来 描述任意对象的数据，也 负责 页面展示，数据保存，数据流转 等 很多操作。主要方法如下：

*  getPojo 将 当前 Entity 对象的数据，转换成 另一个 确切的 类的数据。
*  addEntityObject  在当前 Entity 保存数据的基础之上，在 将另一个Entity保存到当前 Entity 中。
*  getId 获取当前记录的 ID 
*  getValue 根据 key 获取 value .

``` bash
{ 
    "_id" : ObjectId("5c94b40611f218203bda40cf"), 
    "domain" : "8f8b93d7-1459-416b-880a-76cf8893dd31", 
    "192168179CDH" : {
        "order_order" : {
            "Order" : {
                "entityName" : "Order", //获取当前节点的值
                "moduleInfo" : "order_order", 
                "connName" : "ModuleControl", 
                "ip" : "192.168.10.27", 
                "domain" : "8f8b93d7-1459-416b-880a-76cf8893dd31"
            }
        }
    }, 
    "id" : "1bc1e113-5285-4422-81f7-5585da6d65ae"
}
```
_比如，上面的数据，如果想 获取 entityName 节点对应的 value ，在 当前Entity ，只需要 用 getValue("192168179CDH.Order.entityName"); 
就可以了。_

***


*  dropValue 移除当前 key-value 。
*  getJson 将 当前 Entity 对象，转换成 一条 String 格式的 JSON 数据。
*  等等 其他很多方法

Entity 主要的实现 子类，是 MongoEntity。

## Table 介绍

__Table 对象的意思，就是字段描述。__

Table 对象，在 模块初始化启动的时候，作为 模块的基本属性 保存到模块中。