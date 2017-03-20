---
title: JavaScript模块系统对决(PK)：CommonJS vs AMD vs ES2015
---

> 了解目前使用的不同JavaScript模块系统，并找出哪些是您的项目的最佳选择。

随着JavaScript开发越来越普遍，命名空间和depedencies更难以处理。开发了不同的解决方案以模块系统的形式来处理这个问题。在这篇文章中，我们将探讨开发人员目前使用的不同解决方案以及他们尝试解决的问题。阅读！

------

## 简介：为什么需要JavaScript模块？

如果你熟悉其他开发平台，你可能有一些概念的*封装*和*依赖*的概念。通常孤立地开发不同的软件，直到先前存在的软件需要满足某些需求。在将其他软件带入项目的时刻，在它和新的代码之间创建依赖关系。由于这些软件需要一起工作，因此它们之间不会出现冲突是很重要的。这可能听起来很小，但是没有某种*封装，*这是两个*模块*相互冲突之前的时间问题。这是C库中的元素之一通常带有前缀的原因之一：

```c
#ifndef MYLIB_INIT_H
#define MYLIB_INIT_H

enum mylib_init_code {
    mylib_init_code_success,
    mylib_init_code_error
};

enum mylib_init_code mylib_init(void);

// (...)

#endif //MYLIB_INIT_H
```

封装对于防止冲突和缓解发展至关重要。

当涉及到依赖关系时，在传统的客户端JavaScript开发中，它们是隐式的。换句话说，开发者的任务是确保在执行任何代码块时都满足依赖关系。开发人员还需要确保依赖关系以正确的顺序满足（某些库的要求）。

以下示例是[Backbone.js的](https://github.com/jashkenas/backbone/blob/master/examples/todos/index.html)示例的一部分。脚本以正确的顺序手动加载：

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Backbone.js Todos</title>
        <link rel="stylesheet" href="todos.css"/>
    </head>

    <body>
        <script src="../../test/vendor/json2.js"></script>
        <script src="../../test/vendor/jquery.js"></script>
        <script src="../../test/vendor/underscore.js"></script>
        <script src="../../backbone.js"></script>
        <script src="../backbone.localStorage.js"></script>
        <script src="todos.js"></script>
    </body>

    <!-- (...) -->

</html>
```

随着JavaScript开发变得越来越复杂，依赖管理可能变得麻烦。重构也受损：在哪里应该更新的依赖关系来维持负载链的正确顺序？

JavaScript模块系统试图处理这些问题和其他问题。他们出生的必要性，以适应不断增长的JavaScript景观。让我们看看不同的解决方案带来的表。

## 一个Ad-Hoc解决方案：显露模块模式

大多数模块系统相对较新。在它们可用之前，特定的编程模式开始越来越多地被使用在越来越多的JavaScript代码：揭示模块模式。

```js
var myRevealingModule = (function () {
    var privateVar = "Ben Cherry",
        publicVar = "Hey there!";

    function privateFunction() {
        console.log( "Name:" + privateVar );
    }

    function publicSetName( strName ) {
        privateVar = strName;
    }

    function publicGetName() {
        privateFunction();
    }

    // Reveal public pointers to
    // private functions and properties
    return {
        setName: publicSetName,
        greeting: publicVar,
        getName: publicGetName
    };
})();

myRevealingModule.setName( "Paul Kinlan" );
```

> 这个例子取自[Addy Osmani的JavaScript设计模式](https://addyosmani.com/resources/essentialjsdesignpatterns/book/)书。

JavaScript范围（至少达到ES2015中`let`的外观）在函数级别工作。换句话说，在函数中声明的任何绑定都不能逃避它的作用域。正是由于这个原因，揭示模块模式依赖于封装私有内容的函数（像许多其他JavaScript模式一样）。

在上面的示例中，*公共*符号在返回的字典中显示。所有其他声明由包围它们的函数作用域保护。不必使用`var`和立即调用包含私有作用域的函数; 一个命名函数也可以用于模块。

这种模式已经在JavaScript项目中使用了相当长的时间，并且与封装事物相当好。它不做太多关于依赖性问题。正确的模块系统也尝试处理这个问题。另一个限制在于，包括其他模块不能在同一源（除非使用`eval`）。

#### 优点

- 足够简单，可以在任何地方实现（没有库，不需要语言支持）。
- 可以在单个文件中定义多个模块。

#### 缺点

- 没有办法以编程方式导入模块（除了使用`eval`）。
- 依赖需要手动处理。
- 模块的异步加载是不可能的。
- 循环依赖可能很麻烦。
- 很难分析静态代码分析器。

## CommonJS

CommonJS是一个旨在定义一系列规范以帮助开发服务器端JavaScript应用程序的项目。CommonJS团队尝试解决的一个领域是模块。Node.js开发人员原本打算遵循CommonJS规范，但后来决定反对它。当涉及到模块时，Node.js的实现非常受它的影响：

```js
// In circle.js
const PI = Math.PI;

exports.area = (r) => PI * r * r;

exports.circumference = (r) => 2 * PI * r;

// In some file
const circle = require('./circle.js');
console.log( `The area of a circle of radius 4 is ${circle.area(4)}`);
```

> 在一个晚上，当我提到一个令人沮丧的请求一个功能，我认为是一个可怕的想法，Joyent对我说，“忘记CommonJS。它已经死了，我们是服务器端的JavaScript。- [NPM创建者Isaac Z. Schlueter引用Node.js创建者Ryan Dahl](https://github.com/nodejs/node-v0.x-archive/issues/5132#issuecomment-15432598)

在Node.js的模块系统的顶部有以库的形式的抽象，以桥接Node.js的模块和CommonJS之间的差距。为了这篇文章的目的，我们将只显示大致相同的基本功能。

在Node和CommonJS的模块中，基本上有两个元素与模块系统交互：`require`和`exports`。`require`是一个可用于将符号从另一个模块导入到当前作用域的函数。传递给`require`的*参数*是模块的*id*。在Node的实现中，它是目录中模块的名称`node_modules`（或者，如果它不在该目录中，则是它的路径）。`exports`是一个特殊的对象：放在其中的任何东西将被导出为一个公共元素。字段的名称保留。Node和CommonJS之间的特殊区别是以`module.exports`对象的形式出现。在Node中，`module.exports`是被导出的真正的特殊对象，而`exports`只是一个默认绑定的变量`module.exports`。`module.exports`另一方面，CommonJS没有对象。实际的含义是，在节点中，不可能导出完全预构造的对象，而不通过`module.exports`：

```js
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

CommonJS模块的设计考虑了服务器开发。当然，API是同步的。换句话说，模块在源文件中的时刻和它们所需的顺序被加载。

> [“CommonJS模块的设计考虑了服务器开发。TWEET这个 ![img](https://cdn.auth0.com/blog/resources/twitter.svg)](https://twitter.com/intent/tweet?text=%22CommonJS%20modules%20were%20designed%20with%20server%20development%20in%20mind.%22%20via%20@auth0%20http://auth0.com/blog/javascript-module-systems-showdown/)

#### 优点

- 简单：开发人员可以抓住这个概念，而不用看文档。
- 集成了依赖性管理：模块需要其他模块，并按需要加载。
- `require` 可以在任何地方调用：模块可以以编程方式加载。
- 支持循环依赖性。

#### 缺点

- 同步API使其不适合某些用途（客户端）。
- 每个模块一个文件。
- 浏览器需要加载器库或翻译。
- 没有模块的构造函数（Node支持这个功能）。
- 很难分析静态代码分析器。

### 实现

我们已经谈到了一个实现（部分形式）：Node.js.

![Node.js JavaScript模块](https://cdn.auth0.com/blog/jsmodules/Node.js_logo.svg)

对于客户端，目前有两个受欢迎的选项：[webpack](https://webpack.github.io/docs/commonjs.html)和[browserify](http://browserify.org/index.html)。Browserify被明确发展解析节点般的模块定义（多节点程序包工作外的开箱即用的吧！），并捆绑你的代码加上这些模块中携带的所有依赖一个单一的文件中的代码。在另一方面Webpack中被开发用于处理发布之前创建源转换的复杂管道。这包括将CommonJS模块捆绑在一起。

## 异步模块定义（AMD）

AMD是由一群不喜欢CommonJS所采用的方向的开发者组成的。事实上，AMD在开发初期就从CommonJS中分离出来。AMD和CommonJS的主要区别在于它支持异步模块加载。

> [“AMD和CommonJS的主要区别在于它支持异步模块加载。TWEET这个 ![img](https://cdn.auth0.com/blog/resources/twitter.svg)](https://twitter.com/intent/tweet?text=%22The%20main%20difference%20between%20AMD%20and%20CommonJS%20lies%20in%20its%20support%20for%20asynchronous%20module%20loading.%22%20via%20@auth0%20http://auth0.com/blog/javascript-module-systems-showdown/)

```js
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

异步加载是通过使用JavaScript的传统闭包成语实现的：当所请求的模块完成加载时调用一个函数。模块定义和导入模块由相同的函数承载：当模块被定义时，其依赖性被显式化。因此，AMD加载器可以在运行时对给定项目的依赖图的完整图片。因此，可以同时加载彼此不依赖于加载的库。这对于浏览器尤其重要，因为启动时间对于良好的用户体验至关重要。

#### 优点

- 异步加载（更好的启动时间）。
- 支持循环依赖性。
- 兼容性`require`和`exports`。
- 依赖管理完全集成。
- 如果需要，模块可以分割成多个文件。
- 支持构造函数。
- 插件支持（自定义加载步骤）。

#### 缺点

- 句法稍微复杂一些。
- 加载器库是必需的，除非传递。
- 很难分析静态代码分析器。

### 实现

目前最流行的AMD实现是[require.js](http://requirejs.org/)和[Dojo](https://dojotoolkit.org/)。

![Require.js for JavaScript模块](https://cdn.auth0.com/blog/jsmodules/requirejs-logo.svg)

使用require.js非常简单：在HTML文件中包含库，并使用`data-main`属性告诉require.js应该首先加载哪个模块。Dojo有[类似的设置](http://dojotoolkit.org/documentation/tutorials/1.10/hello_dojo/index.html)。

## ES2015模块

幸运的是，ECMA团队背后的标准化JavaScript决定解决模块的问题。结果可以在最新版本的JavaScript标准中看到：ECMAScript 2015（以前称为ECMAScript 6）。结果是语法上愉快的，并且与同步和异步操作模式兼容。

```js
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

> 示例取自[Axel Rauschmayer博客](http://www.2ality.com/2014/09/es6-modules-final.html)

该`import`伪指令可以用于将模块带入命名空间。这个指令，与`require`和`define`不是动态的（即它不能在任何地方被调用）。`export`另一方面，该指令可以用于将元素显式地公开。

静态特性`import`和`export`静态指令允许静态分析器构建一个完整的依赖关系树，而不需要运行代码。ES2015不支持动态加载模块，但草案规范：

```js
System.import('some_module')
      .then(some_module => {
          // Use some_module
      })
      .catch(error => {
          // ...
      });
```

> 实际上，ES2015 [只指定](https://github.com/lukehoban/es6features/issues/75)静态模块装载器[的语法](https://github.com/lukehoban/es6features/issues/75)。实际上，在解析这些指令之后，ES2015实现不需要做任何事情。仍然需要模块加载器，如System.js。提供了浏览器模块加载的草案[规范](https://github.com/whatwg/loader)。

这个解决方案通过集成在语言中，使运行时选择模块的最佳加载策略。换句话说，当异步加载产生好处时，它可以被运行时使用。

> **更新（2017年2月）：**现在有一个[动态加载模块](https://github.com/tc39/proposal-dynamic-import)的[规范](https://github.com/tc39/proposal-dynamic-import)。这是对ECMAScript标准的未来版本的提议。

#### 优点

- 支持同步和异步加载。
- 语法简单。
- 支持静态分析工具。
- 集成在语言（最终支持到处，不需要图书馆）。
- 支持循环依赖。

#### 缺点

- 仍然不支持全部。

### 实现

遗憾的是，没有一个主要的JavaScript运行时在其当前稳定的分支中支持ES2015模块。这意味着在Firefox，Chrome或Node.js中不支持。幸运的是，许多转换器支持模块，并且[polyfill](https://github.com/ModuleLoader/es6-module-loader)也可用。目前，为[Babel](https://babeljs.io/)预设的ES2015 可以毫无问题地处理模块。

![Babel for JavaScript模块](https://cdn.auth0.com/blog/jsmodules/babel.png)

## 一体化解决方案：System.js

你可能会发现自己试图使用一个模块系统远离遗留代码。或者你可能想确保发生了什么，你选择的解决方案仍然可以工作。输入[System.js](https://github.com/systemjs/systemjs)：支持CommonJS，AMD和ES2015模块的通用模块加载程序。它可以与转换器一起工作，如Babel或Traceur，并且可以支持Node和IE8 +环境。使用它是在代码中加载System.js，然后将其指向您的基本URL：

```html
    <script src="system.js"></script>
    <script>
      // set our baseURL reference path
      System.config({
        baseURL: '/app',
        // or 'traceur' or 'typescript'
        transpiler: 'babel',
        // or traceurOptions or typescriptOptions
        babelOptions: {

        }
      });

      // loads /app/main.js
      System.import('main.js');
    </script>
```

由于System.js可以即时完成所有工作，因此使用ES2015模块通常应该在生产模式下的构建步骤中保留给转换器。当不处于生产模式时，System.js可以为您调用转换程序，提供生产和调试环境之间的无缝转换。

## Aside：我们在Auth0使用什么

在Auth0，我们大量使用JavaScript。对于我们的服务器端代码，我们使用CommonJS风格的Node.js模块。对于某些客户端代码，我们更喜欢AMD。对于我们基于React的[无密码锁库，](https://github.com/auth0/lock-passwordless)我们选择了ES2015模块。

喜欢你看到的？[注册](javascript:signup())并开始在您的项目中使用Auth0。

你是一个开发人员，喜欢我们的代码？如果是，[请](https://auth0.com/jobs)立即[申请](https://auth0.com/jobs)工程学位置。我们有一个真棒团队！

## 结论

构建模块和处理依赖性在过去是麻烦的。较新的解决方案，以图书馆或ES2015模块的形式，已经消耗了大部分的痛苦。如果你正在寻找一个新的模块或项目，ES2015是正确的方法去。它将始终被支持，并且使用transpiler和polyfills的当前支持是优秀的。另一方面，如果你更喜欢使用纯ES5代码，那么客户端的AMD和服务器的CommonJS / Node之间的通常分割仍然是通常的选择。不要忘记在下面的评论部分留下你的想法。Hack on！