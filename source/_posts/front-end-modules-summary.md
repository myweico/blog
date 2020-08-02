---
title: 前端模块化-CommonJS、AMD、UMD以及ESM
date: 2020-07-05 23:46:50
tags:
- front-end
---

## 什么是前端模块？
模块是实现特定功能的部件集合，在前端中，模块则对应于实现某功能的函数、逻辑的集合。

通常来说，一个文件就是一个模块，有自己的作用域，可以导入其他模块以及向外暴露自己的函数和变量。

目前前端模块化有CommonJS、AMD、CMD和ESM这几种主要的标准，而UMD则是一种兼容 CommonJS、AMD以及CMD的一种兼容性写法。

<!-- more --->
## CommonJS
`NodeJS` 是 `CommonJS` 规范的主要实现者。`CommonJS`主要实现如下：

```js
// -- todo.js
var todos = [];

function addTodo(item) {
    todo.push(item);
}

function getTodo() {
    return todos;
}

// 通过 module.exports 导出模块
module.exports = {
    addTodo,
    getTodo
};

// -- index.js
// 通过require引入模块
var todo  = require('./todo');

todo.addTodo('read document');
```

### exports 和 module.exports 的区别
exports是对module.exports的初始引用，一旦 module.exports 的引用指向发生改变，则exports 的引用将会失效
```js
// 若module.exports 的引用没有发生改变，则exports和module.exports 同效
exports = {
    addTodos,
    getTodos
}

// module.exports 的引用发生改变，则 exports 失效
modules.exports = {
    addTodos,
    getTodos
}

// 此时exports已失效
exports.removeTodo = function(index) {
    todos = todos.splice(index);
}
```

CommonJS 加载模块是同步的，只有加载完成才执行后面的操作。

对于NodeJS，由于模块都已经是存在本地的了，所以同步加载模块比较快，不用考虑异步加载。

而对于浏览器来说，若模块都是从网络上同步加载，这将阻塞后面内容的渲染，使用同步加载方案的CommonJS就有点不太合适了。所以就有了 AMD、CMD 等异步加载模块的方案。

## AMD 和 requireJS
[AMD](https://github.com/amdjs/amdjs-api/blob/master/AMD.md) (即 Asynchronous Module Definitio)。[requireJS](https://requirejs.org/) 就是实现了 AMD 规范的一个模块加载库。

通过引入 requireJS，我们就不必手动通过 scirpt 标签引入各种库，`requireJS` 会帮我们处理这一切。

### 引入requireJS
```html
<!DOCTYPE html>
<html>
    <head>
        <title>My Sample Project</title>
        <!-- data-main attribute tells require.js to load
             scripts/main.js after require.js loads. -->
        <script data-main="scripts/main" src="scripts/require.js"></script>
    </head>
    <body>
        <h1>My Sample Project</h1>
    </body>
</html>
```
requireJS 加载完后，就会执行 `scripts/main` 的脚本，相关的引入代码可以在  main 文件中编写

### 定义模块
> requireJS 定义模块文档 => https://requirejs.org/docs/api.html#define

AMD 模块使用 `define` 函数定义模块
```
define(moduleName?: String, dependencies?: String[], factory: Function|Object);
```
各参数定义如下：
- moduleName: 模块名
- dependencies: 依赖列表，每个模块引入后，将会传入到 factory 函数中，默认值为["require", "exports", "module"]
- factory: 最后一个参数，定义了模块的具体实现，模块的导出值为函数的返回值或者指定对象

#### 简单模块
若一个模块，只有输出简单的键值对，则只需在define()定义这些键值对就可以
```js
define({
    todos: [
        'read book',
        'read document',
        'runnig'
    ],
    completed: [
        'breakfast'
    ]
})
```

#### 函数模块
若一个模块没有依赖，只需要函数来做一些简单的初始化工作，然后输出模块，那么可以使用一个函数来定义模块
```js
//my/shirt.js now does setup work
//before returning its module definition.
define(function () {
    //Do setup work here

    return {
        color: "black",
        size: "unisize"
    }
});
```

#### 定义带有依赖的模块
比如，定义一个 `todo`的模块，依赖 `jQuery`模块，则模块的定义为
```js
define('todo', ['jQuery'], function($){
    var todos = [];

    function addTodo(item) {
        todo.push(item);
    }
    
    function getTodo() {
        return todos;
    }
    
    // 模块输出
    return {
        addTodo,
        getTodo
    }
})
```

#### 使用require引入模块
也可以使用 `require` 引用外部依赖
```js
define(function(require, exports, module) {
        var a = require('a'),
            b = require('b');

        //Return the module value
        return function () {};
    }
```

### 使用模块
```js
requrie(['todo'], function (todo) {
  todo.addTodo('read document')
})
```


## CMD 和 sea.js
[CMD](https://github.com/cmdjs/specification/blob/master/draft/module.md)(Common Module Definition)，而`sea.js`就是实现 CMD 规范的一个模块加载包。


### 模块定义
```
// 定义一个模块
define(function(require, exports, module) {
  // 加载jquery模块
  var $ = require('jquery');
  // 直接使用模块里的方法
  $('#header').hide();
});
```

原理可以参考 卢勃 在[知乎](https://www.zhihu.com/question/20342350/answer/14828786)上的回答。原话就是

    1. 通过回调函数的Function.toString函数，使用正则表达式来捕捉内部的require字段，找到require('jquery')内部依赖的模块jquery
    
    2. 根据配置文件，找到jquery的js文件的实际路径
    
    3. 在dom中插入script标签，载入模块指定的js，绑定加载完成的事件，使得加载完成后将js文件绑定到require模块指定的id（这里就是jquery这个字符串）上
   
    4. 回调函数内部依赖的js全部加载（暂不调用）完后，调用回调函数
    
    5. 当回调函数调用require('jquery')，即执行绑定在'jquery'这个id上的js文件，即刻执行，并将返回值传给var b

### 与requireJS的区别
`sea.js` 和 `requireJS` 都是在执行回调前加载完所有依赖，区别就是`requreJS` 在加载完依赖后立即执行，而`sea.js`则是执行回调函数的时候，require 依赖的时候时候才执行对应依赖的代码。

执行代码的时间区别不大，除非是很大的库。

## ESM
ES6 提出了 Javascript 标准的模块规范。详细的ESM模块可以查看 [阮一峰老师的 ES6 教程](https://es6.ruanyifeng.com/#docs/module)

### 模块定义
```
var todos = [];

function addTodo(item) {
    todo.push(item);
}

function getTodo() {
    return todos;
}

// 通过 module.exports 导出模块
export default {
    addTodo,
    getTodo
};
```

### 模块的使用
ESM 使用 `import` 导入相关的模块
```js
import todo from './todo';

todo.addTodo('read document');
```

## UMD
UMD（Universial Module Definition)，是一种兼容 `CommonJS`、`AMD`、`CMD`规范的一种写法。

UMD的写法如下
```js
(function(global, factory) {
    if (typeof module === 'object' && typeof module.exports === 'object') {
        // commonJS
        depModule = require('depModule');
        module.exports = factory(depModule);
    } else if (typeof define === 'function' && define.amd)  {
        // amd
        define(['depModule'], factory);
    } else if (typeof define === 'function' && define.cmd) {
        // cmd
        define(function(require, exports, module) {
            var depModule = require('depModule');
            return factory(depModule)
        })
    } else {
        // 引入到全局变量中
        global.umdModule = factory(global.depModule)
    }
})(this, function(depModule) {
    // 这里可以使用depModule
    
    // 模块的导出
    return {
        ...
    }
})
```

## ES6 模块与 CommonJS 模块的差异
参考[阮一峰老师的话](https://es6.ruanyifeng.com/#docs/module-loader#ES6-%E6%A8%A1%E5%9D%97%E4%B8%8E-CommonJS-%E6%A8%A1%E5%9D%97%E7%9A%84%E5%B7%AE%E5%BC%82)，差异主要有两点
- CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
- CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。


## 参考文章
- [AMD 规范](http://shouce.jb51.net/webpack/amd.html)
- [sea.js的同步魔法](http://hexo.wbjiang.cn/sea.js%E7%9A%84%E5%90%8C%E6%AD%A5%E9%AD%94%E6%B3%95.html)