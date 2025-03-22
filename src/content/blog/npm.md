---
author: Kiera
pubDatetime: 2024-04-02T13:05:56.066Z
modDatetime: 2024-04-02
title: 关于npm，你了解多少
slug: npm
featured: false
draft: false
tags:
  - npm
description: 关于npm，你了解多少
---

## 什么是npm

简单来说，npm就是包管理工具，提供给开发人员publish download实用性比较强的包的工具。

## npm干了什么

首先都很清楚的npm install，在npm install的时候，npm 去干了些什么？

npm 2.x之前，在npm install的时候，npm是递归的去下载项目所需要的依赖，将包构造成一棵依赖树。比如A依赖于B包，在npm install的时候就会把B的安装包下载到A/node_module里面。
项目结构大概是这个样子：

```html
├── node_modules │ ├── A │ │ └── node_modules │ │ │ └── B@1.1.0 │ ├── C@1.1.0 │
├── D │ │ └── node_modules │ │ │ └── B@2.1.0 │ │ │ └── C@1.1.0 以此类推...
```

这样很明显有一个缺点，就是B这种共同依赖的东西 会下载很多次。一个项目几十上百个包，相同的B可能会download个五六七八九十次，大量冗余，这就造成了node_modules包都比较大。

npm 3.x后的版本对2版本的问题做了点处理方案，A，C共依赖的B包既然下载那么多次，那我就把它移到跟AC同级上面来，这样AC要用的时候直接去拿，哎～这样不就减少下载次数了嘛。
注意：AC去拿的时候 只能从自己的目录跟父目录去找，比如这里AC的
这个时候的目录结构大概就成这个样子：

```html
├── node_modules │ ├── A │ │ └── node_modules │ │ │ └── B@1.1.0 │ ├── C@1.1.0 │
│ └── node_modules │ │ │ └── B@1.1.0 //
这里只会从自己的目录或者父目录去查是否存在相同版本的安装包。（不会从旁边的（比如A）node_modules里面去找）
│ ├── D │ │ └── node_modules │ │ │ └── B@2.1.0 //
版本号不对应，所以需要重新下载。 │ │ │ └── ~~C@1.1.0~~ //
版本号及包与父级目录已加载好的模块c重复，则跳过该次安装
```

大概原理呢就是 在npm install的时候，下载A的依赖包时，优先将B加载入A的node_module中，等加载到C时，C依赖B，然后去已安装的父级模块或者自己的目录里面去找有没有适合的B，如果版本对应合适，则跳过，否则重新下载C依赖包及其相应版本。

（注意： 包的依赖是按照字母排序的，其中@符号的优先级最高，然后按照排序一层一层的安装，也就是广度优先遍历。）

“解释一下广度优先遍历：对二叉树的一种遍历方法，先遍历根节点，然后在遍历子节点。遍历时的顺序是先遍历左节点再遍历右节点，遍历完之后不往下遍历，而是遍历同级的兄弟节点。从爷爷辈先左后右遍历下来，长辈遍历完了才遍历爸爸辈，爸爸辈的啥七大叔八大姨都遍历完才轮到孩子辈........依次下去”
![在这里插入图片描述](https://img-blog.csdnimg.cn/97100124a9224576893086ac9847f384.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6LGG6Iq95LiN5ZCD6LGG,size_20,color_FFFFFF,t_70,g_se,x_16)
这样优化看起来确实减少了点重复安装，但也还是存在缺陷。比如

```html
├── node_modules │ ├── A@1.0.0 │ │ └── node_modules │ │ │ └── D@1.0.0 │ ├── B │
├── C │ ├── D@2.0.0 │ ├── E │ │ └── node_modules │ │ │ └── D@1.0.0
```

E依赖于D的1.1.0版本，去查找优先安装好的模块时，发现D@2.0.0与之版本冲突，然后E就生气了，索性懒得找了，你有2.0了不起吗？我就要1.0！ 于是...又自己了安装一个1.0版本....当然如果版本对应，E也想省事儿嘛，既然你都有了，我也能用，那我就薅个羊毛，懒得去再下载了。

这样下次其实重复下载的缺陷还是没解决完，只能说相对优化了很多。再后来的后来 有了个pnpm，可以理解为npm plus～～ 刚好解决了这个问题。这里暂时不赘述。

npm install完成之后 除了会生成node_modules文件之外，还会生成两个大家都很熟悉的文件：package.json / package-lock.json。

package.json: 就是一个json文件，对项目的一些配置，其中包括项目名称/版本号/安装依赖及其版本/script脚本命令/代码规范等一些配置。

贴个掘金看到的链接，package.json中每个字段的具体描述意义：
https://juejin.cn/post/7023539063424548872

package-lock.json: 将node_modules中的依赖关系描述成的树状结构，以及锁定所下依赖的精确版本。目的是为了防止多人开发的时候，彼此之间安装的依赖包版本不统一 造成冲突。

![在这里插入图片描述](https://img-blog.csdnimg.cn/115bef615647438db0da19d16fde1686.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6LGG6Iq95LiN5ZCD6LGG,size_20,color_FFFFFF,t_70,g_se,x_16)
（图出自：https://github.com/jkhhuse/imgRepo/issues/4）
