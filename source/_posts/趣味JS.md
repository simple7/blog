---
title: 趣味js
date: 2017-06-01 14:48
tags: [js]
---
最近碰到一些有意思的js问题，随手记录下来。
```
console.log(a); //=> [Function: a]
var a = 1;
function a() {
    return 2
}
console.log(a);  //=> 1
```
上网查询相关资料后发现，`函数声明的优先级高于变量声明的优先级，但不会覆盖变量赋值`。

我再把`function a`换成函数表达式
```
console.log(a); //=> undefined
var a = 1;
console.log(a)	//=> 1
var a = function() {
    return 2
}
console.log(a);  //=> [Function: a]
```
可以看出，函数声明会提前，也就是说函数声明是在浏览器准备执行代码的时候执行的；而函数表达式是在只是单纯的赋值语句，会覆盖原有值；