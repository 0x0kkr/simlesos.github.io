---
title: "JS模块化"
date: 2015-04-09
thumbnail: https://images.unsplash.com/photo-1592861594356-72c5027d89c8?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=934&q=80
categories: FrontEnd
toc: true
tags:
  - JS
---

在 ES6 之前的 js 中，并没有像 java 的 import，C# 的 using，css 的 @import 这种类似的语言支持的模块系统。

<!--more-->

## 模块是什么?

模块要实现的基本功能包块下面三点:

1. 封装实现
2. 暴漏接口
3. 依赖声明

## 早期的无模块

我们一步一步来看,最早期的时候,假如有这么一段代码:

```javascript
//a.js
function random(number) {
  return Math.floor(Math.random() * number);
}

//b.js
var _max = 10; //直接暴漏私有变量
function getNumber() {
  return random(_max + 1);
}
```

这段代码中，b.js 依赖 a.js，但是却没有显式的依赖关系，完全凭程序员的自觉，并且两个 js 文件中的函数都是暴漏在全局环境中。显然这并不能满足我们的需求。

## 对象字面量包装

我们对上面的代码稍作改进

```javascript
//a.js
var a = function(){
    function random(number){
        return Math.floor(Math.random() * number);
    }
    return {
        random: random
    }
}

//b.js
var b = function(){
    var _max = 10;
    function getNumber(){
        return random(_max + 1);
    }
    return {
        getNumber: getNumber;
    }
}
```

这样看起来代码更具有结构性，b.js 中得私有变量 \_max 也成功的被隔离在闭包中，成为私有成员，外界无法访问。并且每个 js 文件只暴漏了一个全局变量。这种模式在 javascript 中被称为模块暴漏。看起来这段代码似乎改善了一些问题，但是这远远没有达到我们的需求，依赖声明问题依然没有解决。

## IIFE

我们需要一个显式的依赖声明，可以通过 IIFE(Immediately Invoked Function Expression)，函数被包含在一对()内，就成为一个表达式，通过在末尾加上另一个 () 就可以立即执行这个函数，这就是 `立即执行函数表达式`，我们将上面的代码改成这种来看看

```javascript
//a.js
var a = (function(){
    function random(number){
        return Math.floor(Math.random() * number);
    }
    return {
        random: random
    }
})();

//b.js
var b = (function(a){
    var _max = 10;
    function getNumber(){
        return a.random(_max + 1);
    }
    return {
        getNumber: getNumber;
    }
})(a);
```

这里我们通过 IIFE 将 a.js 和 b.js 包裹在一个闭包中，我们不需要再去额外的调用 a，b 函数来获取暴漏接口，并在 b.js 中显式的声明了依赖关系，但是依然没有依赖控制，仍然污染了全局变量。

## 命名空间式的模块

我们可以定义一个模块加载器，核心代码如下:

```javascript
var MyModules = (function Manager() {
  var modules = {};
  function define(name, deps, impl) {
    for (var i = 0; i < deps.length; i++) {
      deps[i] = modules[deps[i]];
    }
    modules[name] = impl.apply(impl, deps);
  }
  function get(name) {
    return modules[name];
  }
  return {
    define: define,
    get: get,
  };
})();
```

我们把所有模块都存放在 modules 列表中，并且统一管理依赖关系，使用起来如下:

```javascript
MyModules.defined("a", [], function () {
  function random(number) {
    return Math.floor(Math.random() * number);
  }
  return {
    random: random,
  };
});

MyModules.defined("b", ["a"], function (a) {
  var _max = 10;
  function getNumber() {
    return a.random(_max + 1);
  }
  return {
    getNumber: getNumber,
  };
});

var b = MyModules.get("b");
b.getNumber(); //=> some number
```

熟悉 AMD 的朋友看起来一定不会陌生。这种命名空间的方式显式声明了依赖关系，并且完全消除了对全局环境的污染，但是依赖管理上还是有问题，如果 a， b 模块定义在不同 js 文件中，那么还是需要人为的去控制 a.js 和 b.js 的引用顺序，简单的还好说，如果有十几个复杂的依赖关系，组成一条依赖链，人力根本无法处理如此复杂的关系网

## 模块系统

模块系统最重要的功能就是依赖管理，包含了 `加载`、`分析`、`注入`、`初始化`这四个最主要的功能，同时不同的模块系统有着不同的模块写法。

### CommonJS

CommonJS 主要被用作非浏览器环境中的模块化，但是通过 [browserify](http://browserify.org/) 或者 [webpack](https://webpack.github.io/) 也可以在浏览器中使用。 大名鼎鼎的 node.js 中的模块化就使用的 CommonJS。 举个栗子:

```javascript
//a.js
function random(number) {
  return Math.floor(Math.random() * number);
}
exports.random = random;

//b.js
var a = require("/a"); // 依赖声明
var _max = 10;
function getNumber() {
  return a.random(_max + 1);
}
exports.getNumber = getNumber;
```

CommonJS 社区活跃度很高，规范接受度也非常高，是运行时支持的，作用域基于文件，一个文件就是一个模块。但是因为使用了同步的 require，没有考虑浏览器环境，在浏览器下无法直接使用。

> 优点:
> 依赖管理成熟可靠
> 社区活跃,规范接受度高
> 运行时支持,模块定义非常简单
> 文件级的模块作用域隔离
> 缺点:
> 不是标准组织的规范
> 同步 require,没有考虑浏览器环境

### AMD

由于 CommonJS 没有考虑异步使用情况，于是就有了 AMD(Asynchronous Module Definition) 规范。比较有名的就是 [requireJS](http://requirejs.org/)，AMD 使用起来和我们上面所讲的命名空间型的模块定义及其相似

```javascript
//a.js
define([], function () {
  function random(number) {
    return Math.floor(Math.random() * number);
  }
  return { random: random };
});

//b.js
define(["/a"], function (a) {
  var _max = 10;
  function getNumber() {
    return a.random(_max + 1);
  }
  return { getNumber: getNumber };
});
```

虽然写法上很相似,但是区区十几行的命名空间模块显然无法和 AMD 相提并论， 最大的区别就是依赖管理。

另外 AMD 还有种 Simplified CommonJS wrapping 的写法，可以和 CommonJS 的写法很相似

```javascript
define(function (require, exports) {
  var a = require("./a");
  var _max = 10;
  function getNumber() {
    return a.random(_max + 1);
  }
  exports.getNumber = getNumber;
});
```

这种写法其实还是依赖前置(通过正则匹配等方式)，并不是类似于 Seajs 的 CMD 那样依赖后置，只是写法上的类似。

> 优点:
> 依赖管理成熟可靠
> 社区活跃,规范接受度高
> 专为异步 IO 环境打造,适合浏览器环境
> 支持类似 Commonjs 的书写方式
> 通过 load plug 可以支持加载非 js 资源
> 成熟的打包工具,并可结合插件使用
> 缺点:
> 模块定义繁琐,需要额外嵌套
> 只是库级别的支持,需要引入额外库
> 无法处理循环依赖
> 无法实现条件加载

## JS 语言级别的模块化

新的 ES 规范中,出现了 JS 语言级别的模块化:

```javascript
//a.js
export default random(number){
    return Math.floor(Math.random() * number);
}

//b.js
import a from 'a';
var _max = 10;
export default function getNumber(){
    return a.random(_max + 1);
}
```

> 优点:
> 真正的规范,模块化的标准
> 语言级别的关键字支持
> 适应所有 javascript 运行时,包括浏览器
> 支持循环依赖
> 缺点:
> 规范未达到稳定级别
> 基本还没有浏览器支持

## 总结

我们首先引入了 IIFE，然后加入了命名空间，但是核心的依赖分析和注入没有实现，于是出现了 AMD， Commonjs， ES6 这些新秀，他们之间也可以通过一些库来实现相互转换，比如 [system.js](https://github.com/systemjs/systemjs)，按需使用。 模块化使得 js 具备开发大型应用的能力，而不是停留在小脚本程序的阶段，当然也离不开标准的制定。
