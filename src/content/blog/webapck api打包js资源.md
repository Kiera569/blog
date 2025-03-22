---
author: Kiera
pubDatetime: 2024-04-01T13:05:56.066Z
modDatetime: 2024-04-01
title: 【webpack】使用webpack api打包一个最简单的js资源
slug: webpack源码
featured: false
draft: false
tags:
  - webpack
description: 如何使用webpack api打包一个最简单的js资源？
---

1. 简单了解一下webpack

> 本质上，webpack是一个用于现代javascript应用程序的静态模块打包工具。当webpack处理应用程序时，它会在内部从一个或多个入口点构建一个依赖图，然后将项目中所需要的每一个模块组合成一个或多个bundles，它们均为静态资源，用于展示你的内容。2. 依赖图

    每当一个文件依赖另一个文件时，webpack都会将文件视为直接存在依赖关系，这使得webpack可以获取非代码资源，如images或web字体等，并会把它们作为依赖提供给应用程序。

3. 什么是webpack cli？
   通过命令行的方式调用webpack，被称为webpack cli.
4. 当我们在终端执行webpack命令时发生了什么
   1. 调用执行webpack包对应的bin目录下的webpack.js文件，紧接着再调用webpack-cli包
   2. 调用执行webpack-cli包对应的bin目录下的cli.js文件，然后再继续调用webpack包。
   3. 调用执行webpack包的API进行打包，如create-react-app的react-scripts：

```javascript
// create-react-app 通过调用 webpack api 来打包
function build(previousFileSizes) {
  console.log("Creating an optimized production build...");
  const compiler = webpack(config);
  return new Promise((resolve, reject) => {
    compiler.run((err, stats) => {
      let messages;
      //  ...其他操作
    });
  });
}

// 其他操作：这里可以通过stats对象拿到打包后所有资源体积以及打包时间。（可用于脚手架日志构建）
```

5. webpack api简介
   使用webpack api: 将webpack.config.js的配置作为参数传递给webpack函数即可。
   使用示例：

```javascript
const webpack = require("webpack");
const compiler = webpack({
  // webpack的诸多配置置于此处
  entry: "./index.js",
});
compiler.run((err, stat) => {
  // 通过stat对象获取关于构建时间及资源等信息
});
```

示例： 使用webpack api打包一个最简单的js资源。
![在这里插入图片描述](https://img-blog.csdnimg.cn/06c0643580ee41dca47b8b292ada42ef.png)
代码如下：

1. 新建一个index.js文件，写入部分内容

```javascript
const sum = (a, b) => {
  return a + b;
};
const result = sum(1, 2);
console.log(result);
```

2. 新建一个test.js文件，写入部分内容。

```javascript
const minus = (a, b) => {
  return a - b;
};

const resultMinus = minus(2, 1);
console.log(minus);
```

3. 新建build.js文件，使用webpack api打包js资源

```javascript
const path = require("path");
const webpack = require("webpack");

function wrapBuild() {
  return webpack({
    entry: {
      index: "./index.js",
      test: "./test.js",
    },
    mode: "none",
    output: {
      path: path.resolve(__dirname, "build"),
      filename: "[name].js",
    },
  });
}

wrapBuild().run((err, stat) => {
  const startTime = stat.startTime;
  const endTime = stat.endTime;
  console.log("构建时间: ", endTime - startTime);
});
```

4. 命令行执行node ./build.js 进行打包。
5. 查看打包产物
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/fafe9f3bd4eb4543a517776ac86de40d.png)
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/7d4b9b6e11364e34931be3e8e9ad52a5.png)
