---
title: JS模块小结
---

一直理不清AMD, COMMONJS, UMD, ES6这些JS模块的区别，这几天一直在看相关内容。
这篇文章源于[Sebastián Peyrott博文](https://auth0.com/blog/javascript-module-systems-showdown/)。
<!-- more -->

当Js开发变得越来越普遍，命名空间和依赖变得越来越难处理，模块系统的出现就是想对这些问题进行解决。在这篇文章里，我们将探讨当前不同模块系统和其能够解决的问题。

## 介绍：为什么我们需要JS模块？

如果你熟悉其他开发平台，你应该注意到*封装*和*依赖*这两个概念。一般来说，不同板块的软件是分开独立开发的，最终根据需求使用特定的板块。这时项目将引入其他模块，项目和其他代码的依赖就此产生。因为这些软件模块需要一起执行，保证它们之间不产生冲突是非常重要的。这听起来觉得很细小，但没有封装模块间产生冲突是早晚会发生的事情。这也是C语言依赖里的元素通常会使用一个前缀的一个原因。

封装对于防止冲突和缓慢开发是十分重要的。

当讲到依赖，在传统的客户端JS开发这是隐含的。换句话说来说，开发者的工作就是保证依赖能在任何代码段中能够成功执行。开发者同时也需要保证依赖是在对的顺序下被满足的（对某些特定库的需求）。

当JS开发变得越来越复杂时，依赖管理将变得累赘。重构代码也变得受妨碍：应该将新增的依赖以什么样的顺序导入保证载入链的正确呢？

JS模块系统就是为了解决这些问题。他们的诞生为了满足日益增长的JS环境。

## 一个特定解决办法：揭示模块模式

大多数模块系统都是最近才出现的。在他们出现前，更多JS代码使用一个特定的编程模式：揭示模块模式。

JS作用域（至少到ES2015 `let` 出现前）是在函数级别的。换句话说，任何声明在函数内的绑定都无法超过其作用域。正因如此，揭示模块模式依靠函数来封装私有内容（和很多其他JS模式一样）。

这个模式在很长时间都被用于JS项目并很好解决了封装的问题，但它并没有解决依赖问题。合适的模块系统应该也解决依赖问题。此外，这个模式还存在一个限制：其他模块不能被用于同一个源问题（除非使用 `eval`）。

** 优点：**
- 足够简单到应用到任何地方（不需要库或语言支持）
- 一个文件可以定义多个模块

** 缺点：**
- 不能通过代码方法导入模块（除非使用`eval`）
- 依赖需要手动处理
- 不能异步载入
- 循环依赖变得十分复杂
- 难以分析静态代码分析器

## CommonJS

CommonJS定义了一系列服务端JS应用开发的规范。模块是CommonJS团队致力于解决的其中一个方面。Node.js开发者期初是按照CommonJS规范开发的但后来决定反对它。当说到模块，Node.js的实现一般是这样的：

```
// In circle.js
const PI = Math.PI;
exports.area = (r) => PI * r * r;
exports.circumference = (r) => 2 * PI * r;

// In some file
const circle = require('./circle.js');
console.log( `The area of a circle of radius 4 is ${circle.area(4)}`);
```

> 在Joyent的一个晚上，当我提到我要加一些我知道是不好的滑稽的需求时，他跟我说：“忘记CommonJS，那是死的，我们是服务端JS。”
> -[NPM创始人Isaac Z. Schlueter引用Node.js创始人Ryan Dahl的话](https://github.com/nodejs/node-v0.x-archive/issues/5132#issuecomment-15432598)

有很多基于Node.js模块系统建立Node.js模块和CommonJS桥梁的库。基于本文的目的，我们仅介绍大致相同的一些基本特征。

在Node和CommonJS模块里，有两个重要的元素用于模块系统交互：`require`和`exports`。`require`是一个用于将其他模块里元素导入到当前作用域的函数。传入`require`的参数是模块的id。在Node的实现里，这是模块在`node_modules`目录中的名字（或者不在这个目录，在某路径下）。
`exports`是一个特别的对象：任何放进里面的东西将导出为共有元素。字段名称是保留的。`modules.exports`的形式是Node和CommonJS一个特有区别。在Node里，`module.exports`是一个被导出的真实对象，而`exports`仅为一个被绑定到默认`module.exports`的变量。另一方面，CommonJS是没有`module.exports`对象。
在Node里实际实现是不能不通过`module.exports`而导出预先构造好的对象。

```
// This won't work, replacing exports entirely breaks the binding to
// modules.exports.
exports =  (width) => {
  return {
    area: () => width * width
  };
}

// This works as expected.
module.exports = (width) => {
  return {
    area: () => width * width
  };
}
```

要牢记CommonJS模块是被设计用于服务器开发的。本质上，其API是同步的。换句话说来说，模块是按照其在源文件中顺序被加载的。

** 优点： **

- 简单：开发者可以掌握其概念而不需要看文档
- 依赖管理是集成的：模块请求和加载其他模块按照一定顺序
- `require`可以在任何位置被调用：模块可以程序化加载
- 支持循环依赖

** 缺点： **

- 同步API使得其不适合特定需求（客户端）
- 一个文件一个模块
- 浏览器需要一个加载库
- 模块没有构造函数（但Node支持）
- 难以分析静态代码分析器

### 实现

我们已经讲到一个实现（以实际形式）：Node.js。

客户端开发当前有两种欢迎的选择：[webpack](https://webpack.js.org/)和[browserify](http://browserify.org/index.html)。
Browserify是特定被开发用于解析Node-like模块定义（很多Node包使用它能够马上工作）和将你的代码和来自其他模块的代码捆绑到一个单独文件来实现所有依赖。
Webpack则是被开发用于处理在发布前源文件转换产生的复杂管道。这包含将各种CommonJS模块捆绑在一起。

## 异步模块定义（AMD)

AMD诞生在一组不满CommonJS采用方针的开发者中。实际上，AMD是在CommonJS早期开发分离出来的。AMD和CommonJS的主要区别在于AMD支持异步模块加载。

```
//Calling define with a dependency array and a factory function
define(['dep1', 'dep2'], function (dep1, dep2) {
    //Define the module value by returning a value.
    return function () {};
});

// Or:
define(function (require) {
    var dep1 = require('dep1'),
        dep2 = require('dep2');

    return function () {};
});
```

异步加载可以用在JS传统闭包习惯中：当请求的模块被成功加载后一个函数才被调用。同一个函数可以实现模块定义和导入模块：当一个模块被定义其依赖也变得明确。
一个AMD加载器因此可以在运行环境仲有一张复杂的依赖图。对于启动时间作为良好用户体验的浏览器来讲，这是特别重要的。

** 优点： **

- 异步加载（更好的启动时间）
- 支持循环依赖
- 兼容`require`和`exports`
- 完全继承依赖管理
- 支持构造函数
- 支持插件（客户载入阶段）

** 缺点： **

- 语法稍微更复杂
- 需要加载库
- 难以分析静态代码分析器

### 实现

[require.js](http://requirejs.org/)和[Dojo](https://dojotoolkit.org/)是当前AMD最受欢迎的实现。

require.js的使用相当简单：在HTML文件中包含库和使用`data-main`属性告诉require.js先下载哪个模块。Dojo有[类似的使用](http://dojotoolkit.org/documentation/tutorials/1.10/hello_dojo/index.html)。

## ES2015模块

幸运地，JS标准背后的ECMA团队决定解决模块问题。最新一版的JS标准：ECMAScript 2014中可以看到成果（之前被称为ECMASript 6）。这个成果语法简单并同时兼容同步和异步操作模式。

```
//------ lib.js ------
export const sqrt = Math.sqrt;
export function square(x) {
    return x * x;
}
export function diag(x, y) {
    return sqrt(square(x) + square(y));
}

//------ main.js ------
import { square, diag } from 'lib';
console.log(square(11)); // 121
console.log(diag(4, 3)); // 5
```

`import`指令可以用于把模块放在命名空间里。相对`require`和`define`指令，`import`并不是动态的（它不能被调用在任何位置）。`export`指令则可以让元素公有。

`import`和`export`的静态特征可以使得静态分析器不用运行代码而建立完整的依赖树。ES2015不支持动态加载模块，但一份草稿规范可以：

```
System.import('some_module')
      .then(some_module => {
          // Use some_module
      })
      .catch(error => {
          // ...
      });
```

>实际上，ES2015[仅描述了静态模块加载器的语法](https://github.com/lukehoban/es6features/issues/75)，ES2015的实现并不需要在解析这些指令后做任何事。像System.js这种的模块加载器一样是需要的。有一份草案规范支持[浏览器模块加载](https://github.com/whatwg/loader)。

这个办法通过整合到一个语言里使得运行时能选择最佳加载模块策略。换句话说说，当异步加载能获得好处，其就可以在运行时使用。

>** 更新（2017年2月）**：现在有[动态加载模块的规范](https://github.com/tc39/proposal-dynamic-import)。这是一个未来发布的EXMASript标准提案。

** 优点： **

- 支持异步和同步模块加载
- 语法简单
- 支持静态分析工具
- 整合到一个语言里（最终在任何位置被支持，不需要库）
- 支持循环依赖

** 缺点： **

- 仍然不能在任何位置被支持

### 实现

不幸的是当前并没有主流的JS运行环境稳定版支持ES2015模块。这意味着这不在Firefox, Chrome或Node.js被支持。幸运的是




