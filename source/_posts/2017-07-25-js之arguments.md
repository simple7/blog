---
title: js之arguments
date: 2017-07-27
tags: [js]
---

> 最近突然想到以前看到一篇文章讲过arguments的一些属性，决定在重温一遍

## arguments
在MDN的解释是一个类似数组的对象, 对应于传递给函数的参数，在控制台输出可以看出其实就是一个形如：
```
{
    0: 'a',
    1: 'b',
    length: 2
 }
```
的对象，不过除此之外它还有callee和caller两个属性，在下节描述。
关于arguments这个对象，常见的操作就是将其转化为数组，这里有两种方法都能将其转化为数组：
```
	//第一种，使用Array原型方法slice
	Array.prototype.slice.call(arguments)
    //第二种 ES6新出现的方法
    Array.from(arguments)
```
## 关于callee 和 caller
其实这两个属性我几乎没用到过，但保不齐哪天需要排上用场呢

### 1. callee
当函数被调用时，它的arguments.callee对象指向自身，也就是对自己的一个引用，这个属性我认为基本用不上

### 2. caller
在一个函数调用另一个函数时，被调用函数会自动生成一个caller属性，指向调用它的函数对象
例如：
```
function testCaller() {  
    var caller = testCaller.caller;  
    console.log(caller); 
    /** 输出内容
        function aCaller() {  
            testCaller();  
        }
    **/
}  
  
function aCaller() {  
    testCaller();  
}  
  
aCaller();
```