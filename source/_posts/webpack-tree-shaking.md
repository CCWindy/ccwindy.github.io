---
title: Webpack打包方式
date: 2018-08-23
tags: Webpack
---
Webpack内置支持使用Tree Shaking方式对程序代码进行打包压缩。
Tree Shaking指移除JS应用程序中未使用的代码的技术，通过Tree Shaking我们可以减少程序代码体积，减少代码解析运行时间等进而达到优化应用程序的目的。
本文将使用一个小例子对Webpack在开发模式及生产模式下的两种打包结果进行对比分析，希望能对Webpack的打包方式有一个更深入的理解。

<!-- more -->

Tree Shaking 基于 ES6 模块系统的静态结构特性，通过对代码中 <code>import</code> 和 <code>export</code> 关键字的遍历，将模块分为了被使用和未使用两部分。
但需要注意 Tree Shaking 仅作用于 ES6 模块结构，如果是 CommonJS 规范 (require) 则无法使用。

我们可以使用 Webpack 对以下两个文件进行打包：
```js
// index.js
import {sum, subtract} from './calculate';
let result1 = sum(1, 2);
let result2 = subtract(1,2);
console.log(result1);
```
```js
// calculate.js
export function sum(a, b) {
    return a + b;
}
export function subtract(a, b) {
    return a - b;
}
export function multiply(a, b) {
    return a * b;
}
```
### Webpack 的开发模式
当 webpack.config.js 中的 <code>mode</code> 为 <code>development</code> 时， Webpack 并不进行代码压缩，注意默认情况下 <code>mode</code> 为 <code>production</code> 。
打包后的 bundle 文件如下（省略了 Webpack 自定义模块加载代码，在每个 bundle 文件中都是一样的）：
```js
// ... 
/******/ ({
    /***/ "./src/calculate.js":
    /*!**************************!*\
      !*** ./src/calculate.js ***!
      \**************************/
    /*! exports provided: sum, subtract, multiply */
    /***/ (function(module, __webpack_exports__, __webpack_require__) {
        "use strict";
        __webpack_require__.r(__webpack_exports__);
        /* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "sum", function() { return sum; });
        /* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "subtract", function() { return subtract; });
        /* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "multiply", function() { return multiply; });
        function sum(a, b) {
            return a + b;
        }
        function subtract(a, b) {
            return a - b;
        }
        function multiply(a, b) {
            return a * b;
        }
        /***/ }),
    /***/ "./src/index.js":
    /*!**********************!*\
      !*** ./src/index.js ***!
      \**********************/
    /*! no exports provided */
    /***/ (function(module, __webpack_exports__, __webpack_require__) {
        "use strict";
        __webpack_require__.r(__webpack_exports__);
        /* harmony import */ var _calculate__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./calculate */ "./src/calculate.js");
        let result1 = Object(_calculate__WEBPACK_IMPORTED_MODULE_0__["sum"])(1, 2);
        let result2 = Object(_calculate__WEBPACK_IMPORTED_MODULE_0__["subtract"])(1,2);
        console.log(result1);
        /***/ })
    /******/ });
```
尽管 multiply 方法并未使用， subtract 方法结果也没被使用，这两个方法还是打包进了最终的 bundle 文件中了。可见此模式下 Webpack 并没使用 Tree Shaking 进行模块打包压缩。

### Webpack的生产模式
当 webpack.config.js 中的 <code>mode</code> 为 <code>production</code> 时， Webpack 自动使用 <code>UglifyJsPlugin</code> 进行代码压缩。
打包后的 bundle 文件如下（省略了 Webpack 自定义模块加载代码，在每个 bundle 文件中都是一样的）：
```js
// ... 
([function (e, t, r) {
    "use strict";
    r.r(t);
    let n = 1 + 2;
    console.log(n)
}]);
```
production 模式下的 bundle 文件精简了很多，也使用了代码混淆方式对变量名进行压缩。更重要的是 Webpack 将多个模块文件包裹在一个闭包里而不是对每个模块文件进行闭包包裹。
这消除了方法的声明定义与调用并直接输出结果，大大减少了代码量。

### Tree Shaking 仅对顶层内容进行处理
我们可以看以下例子：
```js
// index.js
import {Calculate} from './calculate';
let calculate = new Calculate();
let result1 = calculate.sum(1, 2);
let result2 = calculate.subtract(1, 2);
console.log(result1);
```
```js
// calculate.js
export class Calculate {
    sum(a, b) {
        return a + b;
    }
    subtract(a, b) {
        return a - b;
    }
    multiply(a, b) {
        return a * b;
    }
}
```
此例子中我们将原来导出的三个方法变为了一个类的方法并对这个类进行导出调用，并使用 <code>mode</code> 为 <code>production</code> 模式对代码进行 Tree Shaking 打包压缩。
打包后的 bundle 文件如下（省略了 Webpack 自定义模块加载代码，在每个 bundle 文件中都是一样的）：
````js
([function (e, t, r) {
    "use strict";
    r.r(t);
    let n = new class {
        sum(e, t) {
            return e + t
        }

        subtract(e, t) {
            return e - t
        }

        multiply(e, t) {
            return e * t
        }
    }, u = n.sum(1, 2);
    n.subtract(1, 2);
    console.log(u)
}]);
````
虽然我们并没有调用类中的 multiply 方法，但整个类中所有方法都被一并打包进 bundle 文件中了。