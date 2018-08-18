---
title: source map 学习记录
---

问题源于在chrome调试网页想改某元素样式时，发现修改的样式是在style.scss文件上的，然而检查当前元素所在html文件，仅引用了styleA.css文件，并没发现其对styleA.scss文件及没有不同名的style.scss文件的引用，这让我觉得十分怪异。

<!-- more -->

通过整个项目检索发现，styleA.scss引用了style.scss。而在styleA.css.map文件中，有一行代码将styleA.scss和style.scss连在一起。

<code>
"sources": ["styleA.scss","style.scss"]
</code>

## 究竟什么是 css.map文件呢？

### css.map文件的产生

原来在使用scss命令来将scss文件转译为css文件时会同时生成一个同名后缀为css.map的文件。

### css.map文件的作用

很多开发者是通过CSS预处理器如Sass, Less 或 Stylus 生产CSS样式文件的。因为CSS文件是被生成的，直接修改CSS文件并不那么有效。
对于支持CSS.map的CSS预处理器，DevTools是允许你直接在Sources版面修改预处理源代码文件并直接在页面上看到修改效果而不需要离开DevTools或刷新页面。当你查看某元素的样式是被生成的CSS文件提供时，Element版面将提供链接到预处理源文件，而非被生成的.CSS文件。
详情可见[chrome开发工具](https://developers.google.com/web/tools/chrome-devtools/javascript/source-maps?utm_source=dcc&utm_medium=redirect&utm_campaign=2016q3)

在生成的styleA.css文件结尾，可以看到这样一行，浏览器通过这行信息可以找到对应的css.map文件。
<code>
/*# sourceMappingURL=styleA.css.map */
</code>

## 类似的js.map文件

其实现在开发除了会使用预处理生产CSS文件外，很多页面是引用的JS也是被生成的，生产方式可能是其他语言如TypeScript, CoffeeScript的转译，可能是经过压缩，也可能是多个JS文件的合并。
同样的经过转换的JS文件和源文件之间也存在map.css信息文件，其记录的对应信息可以让开发者直接在chrome开发者模式下直接调试源文件。

在生成的main.js文件结尾，也可以看到类似的一行。
<code>
/*# sourceMappingURL=main.js.map */
</code>
