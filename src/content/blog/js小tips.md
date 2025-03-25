---
author: Kiera
pubDatetime: 2024-04-02T13:05:56.066Z
modDatetime: 2024-04-02
title: js小tips
slug: js
featured: false
draft: false
tags:
  - javascript
description: js小tips：通过两个数组获取新的数组对象值
---

小tip~ 感觉经常用 随手记一下 也比较简单。
实现要求：

```javascript
 const newData =  [{a: 3},
   {b: 2},
   {c: 1},
   {d: 2},
   {e: 1},]

   const enumObj = {
    a: '属性1',
    b: '属性2',
    c: '属性3',
    d: '属性4',
    e: '属性5',
  };
   通过传入newData，得到以下返回值：
     [
    { name: '属性1', currentValue: 3 },
    { name: '属性2', currentValue: 2 },
    { name: '属性3', currentValue: 1 },
    { name: '属性4', currentValue: 2 },
    { name: '属性5', currentValue: 1 }
  ]

```

实现思路：

1.  先将数组转换为对象
2.  对象属性/值转为数组形式
3.  遍历数组并重新组装对象

代码：

```javascript
const newObj = data => {
  return Object.entries(Object.assign({}, ...data)).map(([name, value]) => {
    return { name: enumObj[name], currentValue: value };
  });
};
```

Iterator
一种标准化的遍历机制，通过 next() 方法逐个访问元素。
核心机制：
迭代协议:Iterable协议（必须包含[Symbol.iterator]方法，返回一个iterator） / Iterator协议（必须包含next()，返回一个{value: xx, done: boolean,}）

实际用途：

1. 统一遍历方式；
2. 惰性求值；
3. 解构赋值与扩展运算符
4. 自定义数据结构遍历

示例：获取斐波拉契数列

```
  const fibonacci = {
    [Symbol.iterator]() {
      let a = 0, b = 1;
      return {
        next() {
          const current = a;
          [a, b] = [b, a + b];
          return { value: current, done: false }; // 无限迭代，永远不会 done: true
        }
      };
    }
  };

  // 取前 5 个斐波那契数
  let count = 0;
  for (const num of fibonacci) {
    if (count++ >= 5) break;
    console.log(num); // 0, 1, 1, 2, 3
  }
```

AsyncIterator
基础概念： AsyncIterator允许支持异步迭代数据,每次调用next()就会返回一个promise，解析后得到下一个值。
优点：使用for await...of调用，可以使异步代码看起来像同步代码，从而避免过多的promise.then()嵌套。
