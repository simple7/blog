---
title: vue入坑指南（一）
date: 2017-04-22 20:12:30
tags: [vue]
---
> 初入前端三大框架之一：vue，以此系列记录学习路上踩过的坑

### 父子组件通信
刚接触vue组件化编程，说实话有点不太适用，深受长久使用jquery操作dom之害，一开始使用vue就踩坑了。

vue的一大重大思想就是推荐我们使用小型、自包含的可复用的组件来构建大型应用，即组件是应用中独立的单元模块。那么问题来了，对于组件中导入其他组件的话，该怎么进行数据绑定和交互。在一开始我看到一些`$emit`，`props`，`ref`这类关键词是有点懵逼的，因为官方文档实在太长了，我是直接安装vue-cli生成项目直接开干的，导致刚开始有点懵，后来一通查询api算是搞明白了。

- **父组件向自组件通信**

    在子组件要显式申明props
    ```
    // 子组件声明
    exprot default {
        props:{
            myMsg:{
                type: String,
                default: ''
            }
        }
    }
    ```
    在父组件调用处,由于html特性是不区分大小写，所以当使用的不是字符串模版，驼峰式命名的prop需要转换为相应的短横线隔开式
    ```
    // 省略组件的导入和注册
    <child my-msg="hello"></child>

    // 也可以给指定值为父组件属性
    <child :my-msg="message"></child>
    export default {
        data (){
            return {
                message:''
            }
        }
    }
    ```
    `.sync修饰符`

    在vue1.x时用来实现父子组件props双向绑定，但发现了一些问题->官方是这么说的，所以在vue2.0时移除了这个修饰符，但在vue2.3时又引入了，不过代表的是另一层含义->只是作为一个编译时的语法糖存在。它会被扩展为一个自动更新父组件属性的 v-on 侦听器。
    例如：

    ```
    <comp :foo.sync="bar"></comp>
    ```
    会被扩展为：
    ```
    <comp :foo="bar" @update:foo="val => bar = val"></comp>
    ```
    当子组件需要更新 foo 的值时，它需要显式地触发一个更新事件：
    ```
    this.$emit('update:foo', newValue)
    ```


- **子组件向父组件通信**

    在子组件中：
    ```
    <template>
        <div @click=doSomething('123') ></div>
    </template>
    ...
    methods: {
        doSomething: function(id){
            this.$emit('do',id);
            ...
        }
    }
    ...
    ```
    在父组件中：
    ```
    <child @do="onDo"></child>

    ...
    methods: {
        onDo: function(id){
            ...
        }
    }
    ...
    ```
