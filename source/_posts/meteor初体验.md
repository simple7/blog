---
title: Meteor初体验
date: 2017-04-10 14:54:33
tags: [meteor]
---

## Meteor文件结构
通常包含`/client`、`/server`、'/public'、'/lib'四个文件夹，当然还有默认的`/.meteor`文件夹-是Meteor存储内部代码的地方，文件结构如下：
>* 在`/client`文件夹下的代码只会在客户端运行；
>* 在`/server`文件夹下的代码只会在服务器运行；
>* 其他代码则会同时运行在服务端和客户端；
>* 静态文件（如：字体，图片等）存放在`public`文件夹下；

**Meteor加载文件顺序：**

 - 在`/lib`下的文件优先加载；
 - 所有以`main.*`命名的文件将会在其他文件载入后载入；
 - 其他文件以文件名字母顺序载入；

## 模板系统
meteor开发的单页应用，只有一个主页面，任何看似页面的切换，实际页面并没有跳转，而是在此页面进行模板替换。故模板是应用中的基础组成部分。
### 1.模板开始
模板的核心组成是一对HTML、JS文件，约定俗成的命名方式是`template_name.html`、`template_name.js`，Meteor的模板引擎是Blaze-简洁、好用的响应式UI库。
Blaze的核心部分包括两部分：

 >- runtime API--响应式API
 >- compiler--负责编译模板

### 2.模板用法
**关于**
    

## Meteor集合
在Meteor中，关键词`var`限制对象的作用域在文件范围内，不用`var`声明则可以作用于整个应用范围内。

## mogodb
### 1.模糊搜索
```
collection.find({name: new RegExp("^.*"+name+".*$")})
```
使用正则查询实现模糊搜索mongo db