---
author: Kiera
pubDatetime: 2024-04-02T13:05:56.066Z
modDatetime: 2024-04-02
title: webpack源码学习之cjs运行时分析
slug: webpack源码学习
featured: false
draft: false
tags:
  - webpack
description: webpack源码学习之cjs运行时分析
---

_准备工作（接上篇文章的示例也可以）：_
**1. 在index.js文件中引入任一js文件**

```javascript
import sum from "./sum";

const result = sum(1, 2);
console.log(result);
```

**2. sum文件**

```javascript
const sum = (a, b) => {
  return a + b;
};

export default sum;
```

**3. build.js文件**

```javascript
const path = require("path");
const webpack = require("webpack");

function wrapBuild() {
  return webpack({
    entry: "./index.js",
    mode: "none",
    output: {
      path: path.resolve(__dirname, "build"),
      filename: "[name].js",
    },
    infrastructureLogging: {
      debug: true,
      level: "log",
    },
  });
}

wrapBuild().run((err, stat) => {
  const startTime = stat.startTime;
  const endTime = stat.endTime;
  console.log("构建时间: ", endTime - startTime);
});
```

**4. 命令行执行node ./build.js 生成打包产物main.js。（截图未完整, 共87行）**
![在这里插入图片描述](https://img-blog.csdnimg.cn/6670e26fae9a4eeaa2ea6e3c86d3f414.png)
**5. 什么是运行时代码？**
通过webpack打包得到的文件(如/build/main.js的骨架代码), 其中包含了一些webpack如何将多个模块集合在一起的代码, 可粗暴理解为webpack runtime 打包过后的代码。可通过node build/main.js可直接调试。
**6. 示例打包js资源后的运行时代码分析：(含注解)**

```javascript
/******/ (() => {
  // webpackBootstrap
  /******/ "use strict";
  /******/ var __webpack_modules__ = [
    ,
    /* 0 */ // ---模块0, 入口模块 可以理解为index.js模块
    /* 1 */ // ---模块1,打包过后的sum.js
    /***/ (
      __unused_webpack_module,
      __webpack_exports__,
      __webpack_require__
    ) => {
      __webpack_require__.r(__webpack_exports__);
      /* harmony export */ __webpack_require__.d(__webpack_exports__, {
        /* harmony export */ default: () => __WEBPACK_DEFAULT_EXPORT__,
        /* harmony export */
      });
      const sum = (a, b) => {
        return a + b;
      };

      /* harmony default export */ const __WEBPACK_DEFAULT_EXPORT__ = sum;

      /***/
    },
    /******/
  ];
  /************************************************************************/
  /******/ // 模块缓存
  /******/ var __webpack_module_cache__ = {};
  /******/
  /******/ // 模块加载器， 模拟实现common.js的require
  /******/ function __webpack_require__(moduleId) {
    /******/ // 根据moduleId从缓存中读取module
    /******/ var cachedModule = __webpack_module_cache__[moduleId];
    // 若缓存中存在该模块，则返回当前的module.exports
    /******/ if (cachedModule !== undefined) {
      /******/ return cachedModule.exports;
      /******/
    }
    /******/ // 缓存中不存在该模块，则去__webpack_modules__里取。
    // 创建一个新的模块,  写入cache对象, key为moduleId, value 为 exports对象
    /******/ var module = (__webpack_module_cache__[moduleId] = {
      /******/ // no module.id needed
      /******/ // no module.loaded needed
      /******/ exports: {},
      /******/
    });
    /******/
    /******/ // 调用包裹函数，计算模块信息，解析出相关模块内容（如:exports）
    /******/ __webpack_modules__[moduleId](
      module,
      module.exports,
      __webpack_require__
    );
    /******/
    /******/ // 返回当前解析过后的module.exports
    /******/ return module.exports;
    /******/
  }

  var __webpack_exports__ = {};
  // 加载入口函数
  (() => {
    __webpack_require__.r(__webpack_exports__);
    /* harmony import */ var _sum__WEBPACK_IMPORTED_MODULE_0__ =
      __webpack_require__(1);
    const result = (0, _sum__WEBPACK_IMPORTED_MODULE_0__["default"])(1, 2);
    console.log(result);
  })();

  /******/
})();
```

**该运行时代码一共做了三件事：**

- \_\_webpack_modules\_\_ : 维护一个所有模块的数组，通过深度优先遍历将其全部转换为AST，以供给后面的包裹函数解析
- \_\_webpack_require\_\_: 手动实现一个模块加载器；优先从缓存列表**webpack_module_cache**中读取模块，并返回其exports内容。如果缓存中没有，则调用包裹函数解析出模块exports等重要内容，存入缓存列表中并返回exports。
- 调用\__webpack_require_(0): 运行入口模块.

**7. 疑问？**

- **如果不对模块进行缓存，会有什么问题？**
  模块内容重复计算，消耗性能。
  每个模块只在第一次引用的时候产生一个对象，后面都是引用该对象，减少代码复杂度。
- **module.exports和exports有什么区别？**
  默认情况下，Node准备的exports变量和module.exports变量实际上是同一个变量，并且初始化为空对象，我们可以把要输出的东西直接加入在这个空对象里面；但是，如果我们要输出的是一个函数或数组，那么，只能给module.exports赋值，给exports赋值是无效的，因为赋值后，module.exports仍然是空对象。
  **结论：**
  如果要输出一个键值对象{}，可以利用exports这个已存在的空对象{}，并继续在上面添加新的键值；
  如果要输出一个函数或数组，必须直接对module.exports对象赋值。
  **_建议直接对module.exports赋值，可以应对任何情况；_**
  详细的可移步阮一峰大佬的文章：https://www.liaoxuefeng.com/wiki/1022910821149312/1023027697415616
