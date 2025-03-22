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
