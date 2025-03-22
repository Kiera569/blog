---
author: Kiera
pubDatetime: 2024-04-02T13:05:56.066Z
modDatetime: 2024-04-02
title: 如何使用js来实现一个发布订阅模式
slug: js-dev
featured: false
draft: false
tags:
  - javascript
description: 如何使用js来实现一个发布订阅模式
---

走流程，先说一下什么是发布订阅模式～

发布-订阅模式:

> 按照专业的话来说:"发布-订阅是一种消息范式,消息的发送者(称为发布者)不会将消息直接发送给特定的接收者(订阅者),而是将发布的消息分为不同的类别,无需了解有哪些订阅者;同理,订阅者也可以对一个类或者多个类有兴趣,但也只关注有兴趣的部分,无需了解存在哪些发布者".

按照流氓的话来说就是: 发布者跟订阅者一起有一个小家,但是两个人从来不交流,也不想见面.有一天.发布者在家里搞了一系列骚操作,移沙发,搬酒柜,贴墙纸...,订阅者晚上下班回去,她只关注自己的房间有没有被搞的乱七八糟,其他不感兴趣的东西 都当作没有看见.

这时候就有流氓要问了,那两个人都不接触的 还过啥日子呀 出家当和尚得了?

要是这样就出家当和尚了,我发布订阅模式不就垮掉了吗.为了让两个人能有点交流.我特意安排了个机器人当中介,

订阅者跟机器人说,我今天买了橘子放在冰箱,谁要是动了我的橘子,你就告诉我,其他事情 你就不用跟我说了,我也不想知道那个逆子干了些啥事儿.然后出门上班了,

赶巧不巧当天发布者嘴馋就偷吃了一个橘子,结果被机器人打小报告了

.订阅者就觉得发布者干的都是人事儿???????? 赶紧也搞了个小动作,把橘子全部拿到房间了.

也不知道你们是专业的还是流氓~~~~~ 反正我是专业的(狗头保命)

再走流程，实现一下

```javascript
class Event {
  // 订阅事件
  on() {}

  // 发布事件
  emit() {}

  // 关闭事件
  off() {}
}

const e = new Event();

e.on("click", x => console.log(x.id));

e.emit("click", { id: 3 });
```

第一步:观察on跟emit的使用方法,我猜里面需要一个对象来维护type跟callback值

第二步我再猜这个对象中维护的type跟callback值要能对照起来,要不然我咋知道你的callback是不是偷的我的callback.

代码如下:

```javascript
class Event {
  constructor() {
    this.events = {};
  }

  on(type, callback) {
    this.events[type] = callback;
  }
  emit(type, args) {
    console.log(this.events); // { click: [Function (anonymous)] }
    this.events[type](args); // 3
  }
}

const e = new Event();

e.on("click", x => console.log(x.id));

e.emit("click", { id: 3 });
```

一不小心就猜对了~~ 就这么简单?你在向peatch, 那万一我有两个click事件呢
比如这样,

```javascript
const e = new Event();

e.on("click", x => console.log(x.id));

e.on("click", x => console.log(x.id));

e.emit("click", { id: 3 });
```

你不该给我打印出两个三吗?

嗯??? 行 我把callback拿个数组装起来不就好了? 问题不大

于是 抓秃三分之一的头发之后...

```javascript
class Event {
  constructor() {
    this.events = {};
  }

  on(type, callback) {
    // 对象里面是否有type这个属性,如果有,直接push回调函数,没有则重新建立.
    (this.events[type] || (this.events[type] = [])).push(callback);
  }
  emit(type, args) {
    console.log(this.events[type]); // [ [Function (anonymous)], [Function (anonymous)] ]
    this.events[type].forEach(element => {
      element(args); // 3  3 ---执行了两次
    });
  }
}

const e = new Event();

e.on("click", x => console.log(x.id));

e.on("click", x => console.log(x.id));

e.emit("click", { id: 3 });
```

我还想让他只执行一次

你.... 好的，没问题。.

然后 我又抓秃了三分之一的头发才想到addEventListener第三个参数有个once.我在需要once的click加个属性应该可以吧? 先试一波~

于是....

```javascript
class Event {
  constructor() {
    this.events = {};
  }
  on(type, callback) {
    // 对象里面是否有type这个属性,如果有,直接push回调函数,没有则重新建立.
    (this.events[type] || (this.events[type] = [])).push(callback);
  }
  emit(type, args) {
    this.events[type].forEach(element => {
      if (element.once) {
        element.listener(args);
      } else {
        element(args);
      }
    });
  }
  once(type, listener) {
    this.events[type] = this.events[type] || [];
    this.events[type].push({ listener, once: true });
  }
}
```

嗯？？？好像有一点小小的问题呀，once意在监听函数只执行一次，但原本这是api帮我们做了的事情，这里要自己实现一下～ 也就是在发布事件的时候，如果存在once，只执行一次发布，然后全部给干掉。

于是 我抓光了所有的头发，有了以下代码：

```javascript
class Event {
  constructor() {
    this.events = {};
  }
  // 监听一个事件
  on(type, callback) {
    // 对象里面是否有type这个属性,如果有,直接push回调函数,没有则重新建立.
    (this.events[type] || (this.events[type] = [])).push({
      listener: callback,
    });
  }
  // 发布一个事件
  emit(type, args) {
    this.events[type].forEach(element => {
      element.listener(args);
      if (type === "once") {
        this.off(type, element.listener);
      }
    });
  }
  // 事件只执行一次
  once(type, listener) {
    this.events[type] = this.events[type] || [];
    this.events[type].push({ listener, once: true });
  }
  // 当传过来的callback相等,则取消该方法.
  off(type, callback) {
    if (this.events[type]) {
      this.events[type] = this.events[type].filter(
        item => item.listener !== callback
      );
    }
  }
}
```

测试一下，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210709102927199.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1Y2t5NTY5,size_16,color_FFFFFF,t_70)
呀～ 秃了呀。。。
