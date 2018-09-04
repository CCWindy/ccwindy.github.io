---
title: JS 闭包
date: 2018-09-03
tags: Javascript
---
 JS 闭包对我来说一直是个迷，然而其又是 Javascript 中一个重要的概念。
 本文为我对 JS 闭包的一点小理解，从 JS 的函数执行过程引入闭包，解释了什么是闭包，并在最后总结几个常用闭包解决的问题。
 
 <!-- more -->
 ### 一般函数的执行过程
 ```js
let count = 0;
function add(num){
    return num + 1;
}
let result = add(count);
console.log(result);
```
上面代码输出 1 ，具体执行过程如下：
1. 全局执行上下文中声明 count 变量并赋值了 1；
2. 全局执行上下文中声明 add 变量并为其分配了一个函数定义，这时函数内的代码不会被执行；
3. 全局执行上下文中声明 result 变量初始值为 undefined；
4. 全局执行上下文中调用了 add 函数，此时会创建该函数的本地执行上下文并压入执行栈，进入函数执行过程：
   1. 根据传入参数，在该函数的本地执行上下文中声明 num 变量，
   该传参 num 变量的值为调用上下文（本情况下为全局执行上下文）中 count 变量的值 0 ；
   2. 执行加法运算，首先在本地执行上下文中找到 num 的值 0 ；
   3. 返回加法运算后的结果 2 ；
5. add 函数调用返回结果 2 赋值给全局执行上下文中的 result 变量，add 函数的本地执行上下文被推出执行栈并被销毁（包括其中的 num 变量）；

重点注意的是调用函数会创建该函数的本地执行上下文，函数调用结束时该函数的本地执行上下文会被销毁，其中使用到的变量集都将被删除。

```js
function add(){
    let count = 0;
    return count + 1;
}
let result1 = add();
let result2 = add();
console.log(result1,result2);
```
以上代码我们很容易可以看到会输出两个 1 ，因为每次调用 add 函数的时候都是一个新的本地执行上下文，使用 count 变量的初始值都为 1 。
然而我们可以通过使用闭包的方式改写以上函数使得其输出值有所改变。

### 使用闭包
```js
function add(){
    let count = 0;
    return function(){
        return count = count + 1;
    }
}
let result = add();
let result1 = result();
let result2 = result();
console.log(result1,result2);
```
上面的代码分别输出 1 2 ，具体执行过程如下：
1. 全局执行上下文中声明了 add 变量并为其分配一个函数定义，这个函数此时不会被执行（同上）；
2. 全局执行上下文中声明了 result 变量其暂时的值为 undefined ；
3. 全局执行上下文中调用了 add 函数，创建了该函数的本地执行上下文并压入执行栈，进入函数执行过程：
    1. 本地执行上下文中声明了 count 变量并赋值为 0 ；
    2. 本地执行上下文中声明了一个匿名函数并将**其定义及该匿名函数的词法环境**返回，这个函数此时不会被执行；
4. 全局执行上下文中将**闭包**赋值给了 result 变量，将 add 函数的本地执行上下文推出执行栈并销毁；
5. 全局执行上下文中声明了 result1 变量其暂时的值为 undefined ；
6. 全局执行上下文中调用了 result 函数，创建了该函数的本地执行上下文并压入执行栈，进入函数执行过程：
   1. 在该函数的**词法环境**中找到 count 变量的值为 **0** ；
   2. 执行加法运算；
   3. 将运算后的结果 1 赋值给该函数的词法环境中的 count 变量并返回；
7. reuslt1 函数调用返回结果 1 赋值给全局执行上下文中的 result 变量，reulst1 函数的本地执行上下文被推出执行栈并被销毁（但不包括其词法环境中的 count 变量）；
8. 全局执行上下文中声明了 result2 变量其暂时的值为 undefined ；
9. 全局执行上下文中调用了 result 函数，创建了该函数的本地执行上下文并压入执行栈，进入函数执行过程：
   1. 在该函数的**词法环境**中找到 count 变量的值为 **1** ；
   2. 执行加法运算；
   3. 将运算后的结果 2 赋值给该函数的词法环境中的 count 变量并返回；
10. reuslt2 函数调用返回结果 2 赋值给全局执行上下文中的 result 变量，reulst2 函数的本地执行上下文被推出执行栈并被销毁（但不包括其词法环境中的 count 变量）；

在这里我们终于出现了闭包的概念了，什么是闭包呢？

### 闭包的概念
在 Javascript 中函数会形成闭包。闭包是由**函数**及**创建该函数的词法环境**组成的。
**这个环境包含了创建这个闭包时其能访问的所有局部变量。**

### 闭包的用途
#### 模拟私有方法
编程语言中如 JAVA 支持私有方法的实现，通过类可以实现面向对象编程。
然而 Javascript 在 ES6 之前并不存在 class 关键字可以创建类，而尽管 ES6 有了类的概念，但依然没有如 private 的关键字声明一个方法是私有的。
使用闭包我们可以在 Javascript 中模拟私有方法，私有方法限制了外部环境对代码的访问，简化出了公共接口部分，避免用户在调用接口时被非核心代码混淆。 
```js
var TransformDate = function (){
    var date;
    function settDate(inputDate){
        date = inputDate ? new Date(inputDate) : new Date();
    }
    function transform(){
        var day = date.getDate() < 10 ? `0${date.getDate()}` : date.getDate();
        var month = (date.getMonth() + 1) < 10 ? `0${date.getMonth() + 1}` : date.getMonth() + 1;
        var year = date.getFullYear();
        return `${year}-${month}-${day}`;
    }
    return{
        value: function(date){
            settDate(date);
            return transform();
        }
    }
};

var transformDate = TransformDate();
console.log(transformDate.value());
console.log(transformDate.value('2018//09/01'));
```
以上代码我们使用闭包的方式实现一个日期转换面向对象编程，将日期转换为 YYYY-MM-DD 格式，上面代码输出为 当天日期的 ( YYYY-MM-DD ) 格式及 2018-09-01。
日期转换函数被封住为外部不可访问的私有函数，外界使用的时候仅需要调用其提供的 value 接口即可实现日期转换。

####  解决某些异步调用函数作用域问题
在 Web 开发中，大部分我们写的 Javascript 代码都是基于事件的，如点击、请求事件等。我们的代码通常作为回调，为响应事件而执行的函数。
```js
for (var i = 1; i <= 5; i++) {
    (function () {
        var j = i;
        setTimeout(() => {
            console.log(j);
        }, 0);
    })();
}
```
以上代码会输出 1 2 3 4 5 。
我们通过引入一个立刻执行函数使得 setTimeout 函数形成了闭包，在函数回调时，闭包保留着创建 setTimeout 函数时 j 变量的值，输出的将是立即执行函数在每次循环后记录的值。

