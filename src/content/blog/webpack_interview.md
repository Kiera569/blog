---
author: Kiera
pubDatetime: 2024-04-02T13:05:56.066Z
modDatetime: 2024-04-02
title: webpack面试题
slug: webpack
featured: true
draft: false
tags:
  - webpack
description: webpack相关面试题整理
---

1.  请说说 webpack 中常见的 loader?（loader 执行顺序：从后往前，从下往上）

    - **css-loader**： 加载 css,解析 import 的 CSS 文件，最终返回 CSS 代码；
    - **postcss-loader**: 使用 PostCSS 加载并转换 CSS/SSS 文件；
    - **style-loader**: 将 css-loader 解析的内容挂载到 html 中；
    - **file-loader**: 可以指定要复制或者存放文件的位置，以及使用版本 hash 命名以获取更好的缓存；在代码中通过相对 URL 去引用输出的文件
    - **url-loader**: 与 file-loader 相似可设置文件阈值，大于则 file-loader 处理，小于则返回 base64 文件编码，减少请求数量
    - **image-loader**: 加载并压缩图片
    - **sourceMap-loader**: 加载额外的 sourceMap-loader
    - **babel-loader**: 将 es6 转换为 es5；
    - **JSON-loader**: 加载 JSON 文件
    - **ts-loader**: 将 ts 转换为 js
    - **awesome-ts-loader**: 将 ts 转换为 js(性能高于 ts-loader)
    - **tslint-loader**: 检查 ts 语法
    - **eslint-loader**: 检查 js eslint
    - **i18n-loader**: 国际化

2.  请说说有哪些常见的 plugin？

    - **html-webpack-plugin**: 在 dist 下生成 html 文件。（依赖于 html-loader）;
    - **clean-webpack-plugin**: 在每次构建项目之前清除/dist文件夹
    - **copy-public-plugin**： 因为 public 下的资源是固定的，直接拷贝到编译后的文件夹引入使用就可以。如：favicon.ico
    - **open-browser-webpack-plugin**: 启动 webpack 自动打开浏览器
    - **terser-plugin**： 支持压缩 es6；(Webpack4)
    - **mini-css-extract-plugin**: 分离 css 文件，提取为独立文件，支持按需加载
    - **UglifyJsPlugin**: 压缩js；
    - **webpack-bundle-analyzer**: 可视化 webpack 文件体积
    - **Dllplugin**: 动态链接库，将第三方库提取出来单独打包
    - **DllReferencePlugin**: 配合 Dllplugin，通过 manifest.json 映射到相关的依赖上去

3.  loader 跟 plugin 有什么区别？

    a. 本质不一样

        loader 是个函数，对函数接收到的内容进行转换，然后再返回转换结果；Loader 的作用是让 webpack 有加载和解析非 js 文件的能力；

        plugin 是个插件，就是对 webpack 能力的一个扩展；

    b. 配置方式不一样

        loader: 在module.rules中配置，作为模块的解析规则，类型为数组；

        plugin: 在plugins中单独配置，类型为数组，每一项都是一个Plugin的实例，参数都通过构造函数传入

    c. 运行时机不一样

        loader: loader为在模块加载时预处理文件，运行在打包文件之前；

        plugin: 在整个编译周期都起作用

4.  babel-loader / babel/preset-env / babel-polyfill / babel-transform-runtime 有什么区别？

    - **babel-loader**: 把 babel 跟 webpack 连接上，相当于是桥梁（此时还未把 ed6 转换成 es5）
    - **babel/preset-env**: 预设，能把 let 翻译成 es5.(但 promise / gennerator 不行)
    - **babel-polyfill**: corejs 包 + regenerator 包, 才真正的支持 promise 语法； （会污染全局变量）
    - **babel-transform-runtime**: 沙盒环境，避免污染全局变量（适用于开发组件库）

5.  webpack 如何优化构建体积跟速度？

    优化构建体积：

         1. 使用UglifyJsPlugin压缩JS代码；
         2. 使用懒加载： import('./'),then() 或者 () => import('./')
         3. 代码分割（分为 css 代码分割 & js 代码分割）
             css 代码分割：mini-css-webpack-plugin（不分割的话则 css in js）；
             js 代码分割： 1. 适用 entry 手动配置 2. 适用 webpack 的 splitchunk 去自动分割；
         4. Tree shaking(只支持 ES Module，不支持 Commonjs)：不打包源代码中的废弃代码（没用到的代码） --- 优化体积 & 降低构建速度
             如： optimization:{usedExports: true, sideEffects: true} // 识别无用代码
         5. 使用 DllPlugin 与 DllReferencePlugin 这两个插件将第三方库进行预编译，这样在打包项目的时候，就不需要将第三方库的代码再次打包

    优化构建速度：
    a. 使用 thread-loader 或 happypack 插件，利用多核CPU进行并行编译，以加快编译速度

    b. 使用 cache-loader 或 hard-source-webpack-plugin 开启缓存，提升二次构建速度。

    c. 使用 resolve.extensions 确定模块扩展后缀，减少Webpack查找文件路径的时间

    d. 在 rules 规则中，设置 exclude 和 include ，尽量减小需要解析和转换的文件数量

    e. 按需引入插件和第三方库，尽量减少第三方库的引用和使用；

    e. 使用 tree-shaking 和 scope-hoisting 优化打包后代码的运行效率；

6.  说一下 webpack 热更新（HMR）的原理？

    webpack-dev-server(WDS): 一个基于 express 的 web server;server 内部调用了 webpack,提供了热更新的功能；

    热更新的核心是客户端从服务端拉取更新后的文件-chunk diff（chunk 需要更新的部分）；

    实际是 WDS 与浏览器之间维护了一个 websocket，当本地资源发生变化时，WDS 会向浏览器发起更新并带上构建时的 hash，让客户端与上一次资源进行对比，客户端对比出差异后会向 WDS 发起 AJAX 请求获取更改内容，这样客户端就可以再借助这些信息向 WDS 发起 JSONP 请求来获取该 chunk 的增量更新

7.  webpack proxy 为什么能解决跨域？

    通过设置 proxy 实际是在浏览器与服务器中添加了一个代理者，本地发送请求的时候，代理服务器响应请求并将该请求发送到目标服务器上，目标服务器响应数据后发送给代理服务器，最终由代理服务器将数据响应给浏览器；

    这是因为：在代理服务器将数据响应给浏览器时，二者同源 不存在跨域；而服务器与服务器之间不会发生跨域，因为跨域行为是浏览器安全策略；

8.  简单说一下 Webpack 构建流程？

    a. 初始化：首先，Webpack会通过命令行或配置文件等方式来接收参数，进而初始化出一个 Compiler 对象，这个对象中包含了Webpack所有配置信息。

    b. 确定入口：根据配置中的 entry 找出所有的入口文件。

    c. 编译模块：从入口文件出发，Webpack会调用相应的loader对模块进行编译转换，并找出模块之间的依赖关系。

    d. 生成chunk：根据依赖关系生成代码块（chunk），一个chunk包含一组模块。

    e. 输出：将每个chunk输出到配置的路径下，生成打包后的代码

9.  使用 webpack 开发时，你用过哪些可以提高效率的插件？

    webpack-dashbord: 更友好的展示相关打包信息

    webpack-dev-server: 本地开发服务器，能够提供 live reload 功能，即当你修改文件之后, 浏览器自动刷新并加载更新后的模块。

    copy-webpack-plugin: 在webpack中复制文件和目录

    webpack-bundle-analyzer: 可视化webpack输出文件的大小，可以优化和减小输出文件。

    webpack-merge: 提取公共配置，减少重复配置代码

    size-plugin: 监控资源体积变化

    HotModuleReplacementPlugin： 热替换

10. babel 原理?

    Babel的工作原理分为三个阶段：解析（parsing）、转换（transforming）和生成（generation):

    a. 解析：将代码转换成 AST;

    b.词法分析：将代码(字符串)分割为 token 流，即语法单元成的数组;

    c.语法分析：分析 token 流(上面生成的数组)并生成 AST，

    d. 转换：访问 AST 的节点进行变换操作生产新的 AST,在最后的代码生成阶段，Babel将转换后的AST重新转化为普通的源代码，同时创建source map

11. 在实际工程中，配置文件上百行乃是常事，如何保证各个 loader 按照预想方式工作?

    可以使用 enforce 强制执行 loader 的作用顺序;pre 代表在所有正常 loader 之前执行，post 是所有 loader 之后执行

12. 写一个 loader/plugin？

    将原文件中的所有console替换为alert的loader:

    ```js
    module.export = function (source) {
      const result = source.replace("console.log", "alert");
      return `module.exports = ${JSON.stringify(result)}`;
    };

    // 使用：在webpack中配置rules
    {
        test: /\.js$/,
        use:[{
            loader: path.resolve(path/alert-loader.js)
        }]
    }
    ```

    简单plugin的代码结构:

    ```js
    class MyWebpackPlugin {
      apply(compiler) {
        // 在 emit 事件发生时执行回调
        compiler.plugin("emit", (compliation, callback) => {
          console.log("my webpack plugin");
          callback();
        });
      }
    }
    // 使用：
    plugins: [new MyWebpackPlugin()];
    ```

13. 分别介绍一下 bundle，chunk，module 的作用是什么

    module：开发中的每一个文件都可以看作是 module，模块不局限于 js，也包含 css，图片等

    chunk: 表示代码块，一个 chunk 可以由多个模块组成;

    bundle: 最终打包完成的文件，一般就是和 chunk 一一对应的关系;bundle 就是对 chunk 进行编译压缩打包等处理后的产出

14. 文件指纹是什么？怎么用？

    文件指纹是打包后输出的文件名的后缀:

    1、hash：和整个项目的构建相关，只要项目文件有修改，整个项目构建的 hash 值就会更改

    2、chunkhash：和 webpack 打包的 chunk 有关，不同的 chunk 会生出不同 chunkhash

    3、contenthash：根据文件内容来定义 hash，文件内容不变，则 contenthash 不变

15. 如何减少代码体积？
    1. tree shaking
    2. 客户端使用uglify/terser进行代码压缩
    3. 删除未使用的代码
    4. 使用webpack的‘chunkhash’来长期缓存文件资源，可以在文件内容发生变化时更改文件名，使得浏览器能够尽可能长时间地缓存和重用代码。
    5. 使用更小的库，比如lodash/moment替换为更小的库或自己实现
    6. 服务端使用gzip/brotli压缩

compress: {
pure_funcs: ['console.log'], // 删除 console.log
},
