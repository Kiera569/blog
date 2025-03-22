---
author: Kiera
pubDatetime: 2024-03-16T13:05:56.066Z
modDatetime: 2024-03-19
title: 简单了解pnpm
slug: pnpm
featured: true
draft: false
tags:
  - pnpm
description: 什么是pnpm? pnpm有什么优点
---

## 什么是pnpm

官网：https://pnpm.io/zh/motivation

performance npm, 为节约磁盘空间并提升安装速度而生的一个包管理工具。

## 特点

创建非扁平化的node_modules文件夹。
在一个项目中，package的一个特定版本将始终只有一组依赖项。（peer是个例外）
功

## 功能

1. **Workspace**：支持创建一个workspace将多个项目合并到一个仓库中，可搭配monorepo使用。（一个workspace的根目录下必须有pnpm-workspace.yaml文件，可能会有.npmrc文件。）
2. **别名（Alias)**: 支持使用自定义名称安装软件包。比如项目中大量使用了lodash，但其中存在一个Bug，影响使用，为此你修复了这个 bug，但 lodash 并没有合并（merge）它。 通常你会直接从你的 fork 仓库中安装修改过的 lodash (git 托管的依赖) 或者修改一下名称做为新包发布到 npm。 如果你使用第二种解决方式，则必须使用新的包名（require('lodash') => require('awesome-lodash')）来替换项目中的所有引用。这个时候就可以使用`pnpm add lodash@npm:awesome-lodash`设置别名来更改项目中所有Lodash的引用。
3. **命令行自动补全**：使用`pnpm install-completion`来设置自动补全。

## 干了什么？

俗话说 没有对比就没有伤害， 先伤害一下npm。
在npx@3之前，npm采用递归的方式来安装依赖，构造一棵挺拔的依赖树。比如这个样子：

```javascript
├── node_modules
│   ├── A@1.0.0
│   │   └── node_modules
│   │   │   └── D@1.0.0
│   ├── B@1.0.0
│   │   └── node_modules
│   │   │   └── D@2.0.0
│   └── C@1.0.0
│   │   └── node_modules
│   │   │   └── D@2.0.0
```

一个依赖包 一个安装路径 干干净净 整整齐齐。
同时也有一个很明显的缺点，就是D这种共同依赖的东西 会下载很多次。一个项目几十上百个包，相同的D可能会download个五六七八九十次，大量冗余，这就造成了node_modules包都比较大，安装速度过慢。
为了解决这个问题，yarn出现了。扁平化node_modules结构，加快了安装速度。npm一看，哼 想法很好但是谁不会呢... 于是浴血奋战了几个通宵 发布了新版本3，大致操作就是：A，C共依赖的B包既然下载那么多次，那我就把它移到跟AC同级上面来，这样AC要用的时候直接去拿，哎～这样不就减少下载次数了嘛。
注意：AC去查找依赖包的时候 只能从自己的目录跟父目录去找

```javascript

├── node_modules
│   ├── A@1.0.0
│   │   └── node_modules
│   │   │   └── D@1.0.0
│   ├── B@1.0.0    // 这里只会从自己的目录或者父目录去查是否存在相同版本的安装包。（不会从旁边的（比如A）node_modules里面去找）
│   ├── C@1.0.0    // 版本号及包与父级目录已加载好的模块c重复，则跳过该次安装
│   └── D@2.0.0   // 版本号不对应，所以需要重新下载。


  或者

├── node_modules
│   ├── A@1.0.0
│   ├── B@1.0.0
│   │   └── node_modules
│   │   │   └── D@2.0.0
│   ├── C@1.0.0
│   │   └── node_modules
│   │   │   └── D@2.0.0
│   └── D@1.0.0


界定条件：广度优先遍历。npm install的时候，将package.json里的依赖按照字母排序，
         其中@符号的优先级最高，然后按照广度优先遍历的安装方式优先将同一层级的模块包及其依赖安装好
         安装新依赖的时候会先去查找优先安装好的模块，有就复用，没有就自己重新安装。

```

这样确实很大程度的减少了重复安装依赖的问题。但查找重复依赖的方式只能从自己的目录或者父目录去查找，还是避免不了幽灵依赖，重复安装的问题。
然后他见缝插针，带着基于符号链接的node_modules来了。

## 小知识扫盲

**幽灵依赖**：依赖包再依赖的东西，即某个包没有在package.json 被依赖，但是用户却能够引用到这个包。

**软链接**：指向源文件的指针，是一个仅仅只有几个字节的单独文件，拥有自己独立的inode。软链接只是对源文件的引用，前者有变化不会影响后者，但后者有变化会影响前者。（可粗暴理解成'桌面快捷方式'）

**硬链接**：与源文件指向同一个物理地址，与源文件共享数据，两者共同拥有一个inode. 删除源文件，硬链接文件不会被删除，可以用来防止文件被误删。（可粗暴理解成'文件另一入口')

_tips: 可通过 ln -s创建一个软链接， ln 创建一个硬链接_

对比npm & pnpm同时安装一个依赖，node_modules里面除了express还会下载很多其他express模块依赖的包（幽灵依赖），但是用pnpm安装的话，能看到就只加载了express模块。如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/16944faef791454693f7e5060ffd820d.png)
那么pnpm加载express的时候 是怎么处理express的幽灵依赖的呢？
![在这里插入图片描述](https://img-blog.csdnimg.cn/f357ca013b4f497ba21292a2b464afea.png)在使用pnpm安装express的时候，会自动生成一个.pnpm文件（虚拟store)，里面存放的就是包的所有子依赖文件。（既有软链接又有硬链接）。

## 找到真实store文件.pnpm-store

摘自官网：

> 实际store在Mac/linux中默认会设置到{home
> dir}>/.pnpm-store/v3；windows下会设置到当前盘的根目录下，比如C（C/.pnpm-store/v3）、D盘（D/.pnpm-store/v3）

![在这里插入图片描述](https://img-blog.csdnimg.cn/6a7c0532dba64fa99c9df0f49d20448d.png)
但是这是别人的路径，不是我们的。这里踩了个小坑，由于我是双系统（window+WSL）,再加上我的项目都是存放在D盘，然后通过命令行进入的Linux系统，所以按照这种找法找到的是Linux下的.pnpm-store（如图），实际要的是windows下面的.pnpm-store。windows下面的.pnpm-store查找方法：

1. 使用stat命令查看文件信息，获取inode等属性。`stat fileName`
2. 使用find根据目前硬链接文件的inode 去查找到实际有用.pnpm-store。（因为.pnpm里面依赖是硬链接到.pnpm-store的，所以我们可以利用已知的硬链接文件的Inode去反向查找到实际.pnpm-store）
   如图：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/fed5348aae6c47bc8d837f94df266388.png)
   去该.pnpm-store里面看一下该文件。预期结果：跟查找文件一样。因为查找文件是该文件的硬链接，共享一个inode，共享一份数据。
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/f26d0dfd94db44d19cfbafd9c91eac30.png)
   果然都是一样的，再试试硬链接特性 修改一下 预期效果：.pnpm-store里面的文件拥有同样修改。
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/105c4c0f5f514872a00ebe7f6ff60016.png)

## 原理

在使用pnpm install安装的时候，第一阶段会生成一个以符号链接创建依赖性嵌套结构的node_modules，其中每个包的文件都是来自内容可寻址的硬链接；
![在这里插入图片描述](https://img-blog.csdnimg.cn/68d8268208514777821da7b5769a3850.png)
第二阶段符号链接依赖项，处理直接依赖关系。
![在这里插入图片描述](https://img-blog.csdnimg.cn/1c70411df9cd4f03b846839f096d7443.png)
在使用pnpm安装的时候，node_modules存放的是依赖项的软链接，而生成的.pnpm目录下存放的既有依赖的硬链接也有一系列幽灵依赖的软硬链接。圈出部分为该依赖的硬链接文件，硬链接到.pnpm-store。由于该包访问子依赖 只能从自身所在目录或者父目录去查找，所以这里访问其子依赖访问的是父级子依赖的软链接（红框外部分）。而这些幽灵依赖硬链接于.pnpm-store
![在这里插入图片描述](https://img-blog.csdnimg.cn/8c63658e124a46ddb1fdf7c2820aff19.png)

## peer依赖的不同处理

使用软链与平铺目录来构建一个嵌套结构。
如果某个package存在peerDependence，那么它可能会在一个项目中有多个不同的依赖集合，去硬链接几次该package。
比如： foo@1.0.0 有两个peers依赖（bar@1.0.0 和 baz@^1），此时的依赖图为：

```javascript
node_modules
└── .pnpm
    ├── foo@1.0.0_bar@1.0.0+baz@1.0.0
    │   └── node_modules
    │       ├── foo
    │       ├── bar   -> ../../bar@1.0.0/node_modules/bar
    │       ├── baz   -> ../../baz@1.0.0/node_modules/baz
    ├── foo@1.0.0_bar@1.0.0+baz@1.1.0
    │   └── node_modules
    │       ├── foo
    │       ├── bar   -> ../../bar@1.0.0/node_modules/bar
    │       ├── baz   -> ../../baz@1.1.0/node_modules/baz
    ├── bar@1.0.0
    ├── baz@1.0.0
    ├── baz@1.1.0
```

如果一个包本身不存在peerDependence，但是它的依赖项存在perrDenpendence，这些依赖会在更高的依赖图中解析，则这个传递package便可在项目中有几组不同的依赖项。比如：a@1.0.0 具有单个依赖项 b@1.0.0。 b@1.0.0 有一个 peer 依赖为 c@^1。 a@1.0.0 永远不会解析b@1.0.0的 peer, 所以它也会依赖于 b@1.0.0 的 peer。如图：

```javascript
node_modules
└── .pnpm
    ├── a@1.0.0_c@1.0.0
    │   └── node_modules
    │       ├── a
    │       └── b -> ../../b@1.0.0_c@1.0.0/node_modules/b
    ├── a@1.0.0_c@1.1.0
    │   └── node_modules
    │       ├── a
    │       └── b -> ../../b@1.0.0_c@1.1.0/node_modules/b
    ├── b@1.0.0_c@1.0.0
    │   └── node_modules
    │       ├── b
    │       └── c -> ../../c@1.0.0/node_modules/c
    ├── b@1.0.0_c@1.1.0
    │   └── node_modules
    │       ├── b
    │       └── c -> ../../c@1.1.0/node_modules/c
    ├── c@1.0.0
    ├── c@1.1.0
```

## 磁盘清理

随着项目越来越大，版本越来越多，历史版本的资源堆积就会导致store目录越来越大，pnpm提供了以下命令来清理缓存。
执行命令'pnpm store prune'，清除未被引用的包。例如上面引用了express@4.18.1的包在后来的某次修改中升级了版本，被更新到了5.0，那么 store 里面的 4.18.1 的express 就就成了个不被引用的包，执行 pnpm store prune 就可以在 store 里面删掉它了

## 一句话简单总结

通过创建非扁平化的node_modules结构，利用软链接硬链接的方式来减少对相同包的重复安装，从而达到既能节省依赖内存又能提高安装速度的效果。
