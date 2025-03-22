---
author: Kiera
pubDatetime: 2024-04-02T13:05:56.066Z
modDatetime: 2024-04-02
title: JS根据对象的属性值进行排序
slug: property sort
featured: false
draft: false
tags:
  - javascript
description: js小tips：根据对象的属性值进行排序
---

遇到了个bug,从后端拿到数据（对象且无序），需要图表x轴展示该数据值a。调用Object.values(),自动排好了序，刚开始没在意，测试的时候就出问题 了...于是就自己重新写了个排序，就记录一下~

**Object.values()**：获取属性值，返回的是一个数组，用该方法获取的属性值有跟for...in一样的迭代顺序。其中for...in的迭代顺序跟执行环境有关系，并不是固定的按照某种（比如从大到小）方式来排序的。

在对整数型数组进行排序时，不建议利用for...in的迭代顺序。因为如果涉及到增删改节点的话，使用for ..in不一定能访问的到操作前后的值，有可能会被忽略。

Object.keys():获取属性，迭代顺序跟for一致(根据index来排序)。

```javascript
var myobj = {
  a: 99,
  b: 67,
  c: 48,
  d: 67,
  w: 34,
  f: 34,
  j: 56,
  s: 54,
};

// 获取属性值并按从大到小排序
let result = Object.values(myobj).sort((a, b) => {
  return b - a;
});
var new_obj = {};
for (let i = 0; i < result.length; i++) {
  // 将原对象中的键值对按照属性值的排序顺序写入新对象
  Object.keys(myobj).map((item, index) => {
    if (myobj[item] === result[i]) {
      new_obj[item] = result[i];
    }
  });
}
console.log(new_obj);
```

运行结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520105848443.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1Y2t5NTY5,size_16,color_FFFFFF,t_70)
