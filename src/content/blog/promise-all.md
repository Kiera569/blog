---
author: Kiera
pubDatetime: 2024-03-16T13:05:56.066Z
modDatetime: 2024-03-19
title: 手动实现一个promiseAll函数
slug: promise-all
featured: false
draft: false
tags:
  - javaScript
description: 手动实现一个promiseAll
---

函数代码如下：

```js
function my_promiseAll(peomiseArr) {
  return new Promise((resolve, reject) => {
    // 判断是否是promise数组。
    if (!Array.isArray(promiseArr)) {
      return reject(new TypeError("arguments should be an Array!"));
    }
    if (promiseArr.length === 0) {
      resolve([]);
    }
    // 记录原始promises中的执行总数
    let promiseNum = promiseArr.length;
    // 记录已完成 Promise 的数量
    let resolveCount = 0;
    // 按照 Promise 的顺序存储结果
    let resultArr = [];
    // 遍历 promises 数组，使用闭包将当前的索引 i 保存到 .then 的回调函数中
    for (let i = 0; i < promiseNum; i++) {
      // 对当前的 promise 执行 .then 操作
      Promise.resolve(promiseArr[i])
        .then(value => {
          // 成功则进入, 已完成的promise数 +1
          resolveCount++;
          // 存储当前执行成功的promise value
          resultArr[i] = value;
          // 若数组中的所有promise执行完成，则resolve所有结果
          if (resolveCount === promiseNum) {
            resolve(resultArr);
          }
          // 失败的话，使用 reject 直接返回错误
        })
        .catch(err => console.log(err));
    }
  });
}
```
