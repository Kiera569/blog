---
author: Kiera
pubDatetime: 2024-04-02T13:05:56.066Z
modDatetime: 2024-04-02
title: dva 入门
slug: dva
featured: false
draft: false
tags:
  - dva
description: dva 入门学习
---

dva = React-router + Redux + Redux-saga

## 数据流向：

​ 内部设置state（model的状态数据），通常为一个对象（可为任何值），仅由dispatch发起的action去更改.

​ 注：同步行为：reducers ---> 更改state （纯函数---同输入必同输出)

​ 异步行为：effect ---> reducers ----> 更改state.

## Models:

​ 操作时保证state为全新对象，无任何引用。

​ 可通过dva().\_store 访问顶部state数据。

## Action：

​ js对象，必须包含type来指定具体行为，由dispatch发起。

​ 注：dispatch在组件connect models后通过props传入

```
dispatch({
  type:"add",
})
```

## Dispatch：

​ function，用于触发action函数（事件派发器）。

```
diapatch({
  type: "add", // model外调用，需要添加namespace
  payload: {}, // 需要传递的信息
})
```

## Reducer:

​ function, 描述如何改变数据（事件）,累积器,数组含有该API，详细可参考MDN

​ 如：利用reduce实现一个1-5的累加

```
[1, 2, 3, 4, 5].reduce((prev, next) => prev + next);
```

​ 在dva中，recuder聚合积累的结果为state新的值，这里的reducer必须是纯函数，每次操作都返回一个全新数据。

Effect:

​ reducer处理同步行为，effect处理异步行为。dva引入redux-saga来做异步流程控制。（采用Generator将异步转成同步写法（async/await底层原理就是generator),从而将effects转为纯函数.

## Subscription:

​ 订阅数据源，根据条件去dispatch需要的action

​ 数据源：当前时间 / 服务器的websocked连接 / keyboard输入 / history路由变化

```
import key from 'keymaster';

app.model({
  namespace: 'count',
  subcriptions: {
    keyEvent({dispatch}) {
      key('enter', () => dispatch({type: 'add'}) )
    }
  }
})
```

## Router:

​ 通过history监听路由url变化，

​ 延申：history通过history.pushState和history.replaceState改变url。

​ 主要api是：

​ history.pushState() : 跳转路由

​ popState event监听路由变化，但无法监听到history.pushState()时的路由变化.

```
import {Router, Route } from 'dva/router';
app.router(({history}) =>
  <Router history={history}>
    <Route path='/' component={HomePage} />
  </Router>
)
```

dva简单应用（不带model）：

```
import dva from 'dva';
const App = () => <div>Hello dva</div>
// 创建应用
const app = dva();
// 注册视图
app.router(() => <App />);
// 启动应用
app.start("#root")；
```

## 数据流图

view ----> dispatch action ----> state更改 ----> view (connect )

![img](https://img-blog.csdnimg.cn/img_convert/d2d9a3512fd74e48ffdd1a1ead82ccac.png)

dva简单应用（带model）：

```
// 创建应用
const app = dva();
// 注册model
app.model({
  namespace: 'count', // 需外部使用，必写
  state: 0,
  // 处理同步---纯函数
  reducers: {
    add(state) { return state + 1 },
  },
  // 处理异步---generator
  effect: {
    *addAfterSecond(action, { call, put }) {
      yield call(delay, 1000); // 执行异步函数
      yield put({ type: 'add' }) // 发出一个Action, 类似dispatch
    }
  },
});
// 注册视图
app.router(() => <App /> );
// 启动应用
app.start("#root");
```

_小例子：写一个列表，包含删除按钮，点击删除按钮延迟1s后删除。_

代码如下：

```
import React from 'react';
import dva, { connect } from 'dva';

// 1. 创建应用
const app = dva();

// 2. 注册Model
app.model({
  namespace: 'list',
  state: [{ id: 1, name: 'dva1' }, { id: 2, name: 'dva2' }],
  reducers: {
    // list--最新state值，data--传参
    delete(list, data) {
      return list.filter(item => item.id !== data.payload.id);
    }
  }
});

// 3. 创建View
const App = connect(({ list }) => ({
  list
}))(function view(props) {
  {
    return (props.list || []).map(item => {
      return (
        <div style={{ display: 'flex', marginTop: '20px' }} key={item.id}>
          <h2>{item.name}</h2>
          <button
            style={{ marginLeft: '20px' }}
            onClick={() => {
              props.dispatch({
                type: 'list/delete',
                payload: item
              });
            }}
          >
            删除
          </button>
        </div>
      );
    });
  }
});

// 4. 注册视图
app.router(() => <App />);

// 5. 启动应用
app.start('#root');
```
