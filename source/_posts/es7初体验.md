
---
title: ES2016初体验
date: 2017-05-18 14:54:33
tags: [es2016]
---

## ES2016的最新特性

`ES2016`(简称es7)在2016年6月已经正式发布了。相比`ES2015(ES6)`,`ES2016`的新特性并不多，所以可能大家也并没有对其作太多的关注，但是并不表示这些新特性就不重要。本文将会对ES2016的部分新特性作一个介绍。

### Array.includes

数组的新方法`includes(element, [fromIndex])`用以检查数组中是否存在`element`元素，然后返回一个布尔值(true/false)。可选的`fromIndex`允许指定开始搜索的位置。

基础用法如下：
```
const elements = ['pig', 'dog', 'cat']
elements.includes('pig')	// => true
elements.includes('horse')	// => false
```
`pig`存在于`elements`数组中，所以`includes`方法返回`true`。而对于`horse`，由于其在`elements`数组中不存在，故返回`false`。

需要注意的是，`includes()`方法必须匹配到严格相等的类型才会返回`true`。也就是说，`includes()`方法在比较的时候相当于`===`。
```
const items = [1, '2', 3]
items.includes(2) 	// => false
items.includes(3) 	// => true
```
当匹配对象的时候，`includes()`方法比较的是对象的引用是否相同。
```
class Person {
  constructor(name) {
    this.name = name
  }
}
const personA = new Person('Linda')
const personB = new Person('Calvin')
const persons = [personA, new Person('Calvin')]
persons.includes(personA) 	// => true
persons.includes(personB) 	// => false
```
在使用可选参数`fromIndex`时的使用方法：
```
const companies = ['baidu', 'alibaba', 'tencent']
companies.includes('baidu', 1) 	// => false
companies.includes('tencent', 1) 	// => true
companies.includes('alibaba', 5) 	// => false
```

`includes()`有一个特别的地方需要注意。我们知道，`NaN === NaN`返回的是`false`。但是，`includes()`方法却能查找数组中的`NaN`。
```
const array = [1, 2, NaN]
array.includes(NaN) 	// => true
```

### 乘方运算符

ES2016中的乘方运算符`**`允许你作乘方运算，这在一些场景中会减少一些比较恶心的`for`循环运算。
```
2 ** 3 	// => 8
6 ** 2 	// => 36
0.8 ** 1 	// => 0.8
```

我们知道，在基本运算（加减乘除）中，有简写的`+=`、`-=`、`*=`和`/=`操作。同样，对于乘方运算符，也有这样的简写方式。

```
let number = 3
number **= 3
number;	// => 27

```

对于乘方运算的一些特殊情况：
```
10 ** 0 	// => 1
NaN ** 0 	// => 1
Infinity ** 0 	// => 1
10 ** NaN 	// => NaN
NaN ** NaN 	// => NaN
```
### async await
当然，还有众所周知的async,await,对于async,await的讲解，我这里推荐阅读[体验异步的终极解决方案-ES7的Async/Await](https://cnodejs.org/topic/5640b80d3a6aa72c5e0030b6)
### 写在最后

目前最新版的`Chrome`浏览器已经完全支持这些新特性，所以你完全可以在不依赖`babel`的情况下使用这些新特性。

当然还有很多新特性，ES5,ES6出来以后我已经完全抛弃了undersource这类库，后续更新其他新玩法。嗯，暂且先这样吧。

