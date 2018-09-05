---
title: JS 继承与原型链
date: 2018-09-05
tags: Javascript
---
像许多编程人员熟悉的 Java，C++ 等语言是基于类的面向对象编程语言，然而在 Javascript 中，并不存在类这种概念，尽管在 ES6 引入了关键字 <code>class</code> ，但这仅仅是语法糖。
本质上 Javascript 是基于原型实现面向对象编程的。本文将对原型链以及其如何实现继承进行梳理。
<!-- more -->

说到 Javascript 的继承，其只有一种结构：对象。
每个 Javascript 对象都有一个私有属性（称为 <code>\_\_proto__</code>）指向这个对象的原型对象，这个原型对象也有一个属性 <code>construtor</code> 指向这个对象的构造函数，这个构造函数也有一个属性 <code>prototype</code> 可以指向这个原型对象。

{% asset_img prototype_relationship.jpg 原型、构造函数和对象的关系 %}

```js
function Shape() {
    this.x = 0;
    this.y = 0;
}
var shape = new Shape();
console.log(shape.__proto__ === Shape.prototype);
console.log(shape.__proto__.constructor === Shape);
```
以上代码会输出两个 true 。

### 基于原型链的继承 
##### 原型链
Javascript 对象有 <code>\_\_proto__</code> 对象指向原型对象，原型对象也有一个 <code>\_\_proto__</code> 对象指向自己的原型对象，层层向上直到一个原型对象的原型对象为 null ，这就是原型链。
当我们试图访问一个 Javascript 对象的属性时，会先在这个对象自身属性中寻找，找不到则在这个对象的原型对象中找，找不到再找这个原型对象的原型对象，层层遍历直到找到或原型对象为 null 。
可以说 Javascript 访问对象的属性时，是沿着原型链去寻找的。
```js
var a = [1, 2, 3];

console.log(a.hasOwnProperty('forEach'));
console.log(a.__proto__.constructor === Array);
console.log(a.__proto__.hasOwnProperty('forEach'));
```
{% asset_img prototype_chain.jpg 原型链 %}

以上代码输出为 false true true 。
a 本身并没有 <code>hasOwnProperty</code> 属性，但 Object 有。a 作为字面量创建的数组对象，其原型链为 a --> Array.prototype --> Object.prototype --> null ，如上图所示。
通过原型链我们可以在 a 中调用 <code>hasOwnProperty</code> 属性，但 <code>hasOwnProperty</code> 只会查询对象本身的属性，并不会在原型链上查询，因此第一行输出为 false 。 
<code>forEach</code> 为数组对象的自身属性，第三行输出为 true 。

#### 继承
```js
function Shape() {
    this.x = 0;
    this.y = 0;
}
Shape.prototype.move = function () {
    this.x++;
    this.y++;
};
function Rectangle(){
    Shape().call(this);
    this.width = 10;
    this.height = 5;
}

Rectangle.prototype = Object.create(Shape.prototype);
Rectangle.prototype.constructor = Rectangle;

var rectangle = new Rectangle();

console.log(rectangle.x);
rectangle.move;
console.log(rectangle.x);
```
以上代码会输出 0 1 。
Rectangle 继承了 Shape 类，首先在 Rectangle 构造函数中调用了 Shape。 
ES5 引入了 <code>Object.create()</code> 方法，使用现有对象提供给新创建对象的原型。在这里创建了新对象，并赋值给了 Rectangle 的原型，使得 Rectangle 的原型指向 shape 的原型。
此外，还需将 Rectangle 的原型对象的构造函数指向 Rectangle 才能完成继承。

#### 继承多个对象
JAVA 中不允许一个子类继承多个父类，但可以通过接口的方式实现多继承。Javascript 中我们可以使用混入的方式实现多继承。
我们继续使用上面的例子
```js
function Shape() {
    this.x = 0;
    this.y = 0;
}
Shape.prototype.move = function () {
    this.x++;
    this.y++;
};
Description.prototype.use = function () {
    this.age++;
};
function Rectangle() {
    Shape.call(this);
    Description.call(this);
    this.width = 10;
    this.height = 10;
    this.name = 'Rectangle';
}

Rectangle.prototype = Object.create(Shape.prototype);
Object.assign(Rectangle.prototype, Description.prototype);
Rectangle.prototype.constructor = Rectangle;

var rectangle = new Rectangle();

console.log(rectangle.x);
rectangle.move();
console.log(rectangle.x);
console.log(rectangle.name);
console.log(rectangle.age);
rectangle.use();
console.log(rectangle.age);
```
以上代码会输出 0 1 0 1 。
<code>Object.assign()</code> 方法可以将所有可枚举属性的值从一个或多个源对象复制到目标对象中。
在 Rectangle 构造函数中，我们不仅调用了 Shape 的构造函数，还调用了 Description 构造函数。
此外，我们除了像单继承那样把 Rectangle 的原型指向了 Shape 的原型，还将 Description 原型对象上的属性复制到了 Rectangle 上了，从而实现 Rectangle 可以访问 Descriptoin 原型对象上的属性。
 





