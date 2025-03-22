---
author: Kiera
pubDatetime: 2024-04-02T13:05:56.066Z
modDatetime: 2024-04-02
title: 【react】使用dom-to-image 将html转为图片并保存
slug: react-dev
featured: false
draft: false
tags:
  - react
description: 使用dom-to-image 将html转为图片并点击下载保存
---

dom-to-image转化核心主要是foreignObject标签，它可以嵌套任何html标签。而SVG元素允许包含不同的XML命名空间，利用实例化XMLSerializer并调用其serializaToString()方法把一个XML文档或者Node对象转为未解析的XML标记的一个字符串，然后再放到forgeinobejct标签里面将其解析成SVG的base64 url。
然后调用其封装好的方法就OK了，先看看效果图：

![上部分为内容，下部分为转出的图片](https://img-blog.csdnimg.cn/20210207133243244.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1Y2t5NTY5,size_16,color_FFFFFF,t_70#pic_center)

代码如下：react-hooks
先下载必要的两个包：

```javascript
yarn add dom-to-image 或 npm install dom-to-image --save
yarn add file-saver 或 npm install file-saver --save
```

```javascript
import React from "react";
import domtoimage from "dom-to-image";
import { saveAs } from "file-saver";
import "./index.css";
// 过滤掉所有<i>元素
function filter(node) {
  return node.tagName !== "i";
}
const DomToImg = () => {
  // 转为png格式的图片
  const getImg = () => {
    const node = document.getElementById("node");

    domtoimage
      .toPng(node)
      .then(defaultUrl => {
        const img = new Image();
        img.src = defaultUrl;
        img.setAttribute("className", "pngImg"); // 方便设置样式
        // 将生成的png图片插入到当前页面
        document.getElementById("export-img").appendChild(img);
        // 手动点击图片下载 自动下载调用saveAs(defaultUrl, '自动保存.png')
        img.addEventListener("click", () => {
          var link = document.createElement("a");
          link.download = "古诗词.png";
          link.href = defaultUrl;
          link.click();
        });
      })
      .catch(() => {
        console.log("error");
      });
  };
  // 生成图片自动下载为png格式（将dom转为二进制再编译下载）
  const getBlobPng = () => {
    const node = document.getElementById("node");
    domtoimage.toBlob(node).then(blob => {
      // 调用file-save方法 直接保存图片
      saveAs(blob, "自动保存.png");
    });
  };
  // 转为Jpeg图片  --- 手动下载（自动下载调用saveAs(defaultUrl, '自动保存.png'))
  const getJpeg = () => {
    const node = document.getElementById("node");
    domtoimage.toJpeg(node, { quality: 0.95 }).then(defaultUrl => {
      var link = document.createElement("a");
      link.download = "下载jpeg.jpeg";
      link.href = defaultUrl;
      link.click();
    });
  };
  // 转为SVG图片---手动下载 （自动下载调用saveAs(defaultUrl, '自动保存.png'))
  const getSVG = () => {
    const node = document.getElementById("node");
    domtoimage.toSvg(node, { filter: filter }).then(defaultUrl => {
      const img = new Image();
      img.src = defaultUrl;
      img.setAttribute("className", "svgImg");
      document.getElementById("export-img").appendChild(img);
      img.addEventListener("click", () => {
        var link = document.createElement("a");
        link.download = "SVG";
        link.href = defaultUrl;
        link.click();
      });
    });
  };
  return (
    <div className="dom-to-img">
      <h1 className="dom-title">html转换为图片</h1>
      <div id="node" className="node-content">
        <p className="node-title">锄禾日当午</p>
        <p className="node-title">汗滴禾下土</p>
        <p className="node-title">谁知盘中餐</p>
        <p className="node-title">粒粒皆辛苦</p>
      </div>
      <div className="my-actions">
        <button onClick={getImg} className="action">
          转为png图片
        </button>
        <button onClick={getBlobPng} className="action">
          自动保存png
        </button>
        <button onClick={getJpeg} className="action">
          自动保存jpeg
        </button>
        <button onClick={getSVG} className="action">
          转为svg图片
        </button>
      </div>
      <div id="export-img" className="my-image"></div>
    </div>
  );
};

export default DomToImg;
```

原理简易代码：

```javascript
<svg>
  <foreignObject x="0" y="0" width="100%" height="100%">
    <div>
      <span>dom转为html</span>
    </div>
  </foreignObject>
</svg>
```
