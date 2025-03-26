---
author: Kiera
pubDatetime: 2024-03-16T13:05:56.066Z
modDatetime: 2024-03-19
title: react 18 新特性
slug: react18
featured: false
draft: false
tags:
  - react
description: react18 有哪些新特性？
---

变更日志：https://reactjs.org/blog/2022/03/29/react-v18.html

## **新版本react更改了些什么**

## ReactDOM.render ----> ReactDOM.createRoot();

react17：

```javascript
import App from "App";
import ReactDOM from "react-dom";

ReactDOM.render(<App />, document.getElementById("root"), () =>
  console.log("render")
);
```

react 18:

```javascript
import ReactDOM from 'react-dom/client';

const App = ({callback}) => {
  return (
    <div ref={callback} >app</div>
  )
}

const root = ReactDOM.createRoot(document.getElementById('root');

root.render(
  <React.StrictMode>
    <App callback={() => console.log("render")}/>
  </React.StrictMode>
)
```

新版本中仍可使用render写法，但推荐使用createRoot，否则无法享受到react18中支持的新特性更改的并发模式带来的好处。react18为严格模式引入了一个新的仅限开发的检查：每当第一次安装组件时，此新检查将自动卸载并重新安装每个组件，并在第二次安装前恢复先前的状态。
**修改目的**
react有三种启用模式：

1. legacy模式： ReactDOM.render(<App />, rootNode); 传统模式是触发的同步的渲染链路。
2. blocking模式： 渐进模式， 过渡到下面的concurrent模式。（很少用）
3. concurrent模式： ReactDOM.createRoot('root').render(<App />)。 并发模式（异步）。
   react内部通过修改fiber.mode这个属性来标识当前处于哪个模式，在执行过程中也是通过判断该属性来区分不同的渲染模式。使用了createRoot才能享受react18新特性。
   **createRoot原理**
   createRoot内部实现核心createRootImpl方法， 创建了一个并发模式的fiberRoot(通过new FiberRootNode生成，fiberRoot通过改变current指针指向来渲染不同的页面)给root，将createHostRootFiber创建的rootFiber(通过new FiberNode生成，与App、div等都有对应的Fiber节点，共同构成一棵Fiber树)绑定到对应的节点的“\_\_reactContainer$'+randomKey属性上。其中randomDomKey通过Math.random..toString(36).slice(2)生成，目的也是为了避免覆盖dom上的原有属性和避免被开发者覆盖。其次是在根节点上监听各种事件，比如click,scroll。
   详细源码解析参考文档：https://zhuanlan.zhihu.com/p/409225031
   _总结： 可在react18中是有老版本注册根节点的方式，emmm....可以但不必要。_

## 自动合并多次状态更新（batching)

批处理：react将多个状态更新分组到一个重新渲染中以避免一些不必要的render以及防止组件渲染仅更新一个状态变量的“half-finish"状态，来获得更优的性能。
比如在一个按钮点击事件中更新两个状态，react会将两个批量分为一个重新渲染。也就是在一次重新渲染中处理两个状态的更新。![在这里插入图片描述](https://img-blog.csdnimg.cn/99a68c5b7dd843daac0831aecf11eabc.png)

但是这种批处理的更新时机并不一致，比如，在获取数据后更新某个state，这时react不会批处理该操作。而是执行两次独立的更新。这是因为react过去只在浏览器事件（比如click）期间进行批量更新，但这里是事件已经被处理之后(也就是请求结束)再去更新state。
react17 执行效果:
![在这里插入图片描述](https://img-blog.csdnimg.cn/b7122121695d468e90983deb40ddb660.png)
react 17 代码如下：

```javascript
import { useState, useLayoutEffect } from "react";
import * as ReactDOM from "react-dom";

function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    console.log("=== click ===");
    fetchSomething().then(() => {
      setCount(c => c + 1); // Causes a re-render
      setFlag(f => !f); // Causes a re-render
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
      <LogEvents />
    </div>
  );
}

function LogEvents(props) {
  useLayoutEffect(() => {
    console.log("Commit");
  });
  console.log("Render");
  return null;
}

function fetchSomething() {
  return new Promise(resolve => setTimeout(resolve, 100));
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

同样的代码，看看react18所有更新自动批处理后的渲染效果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/0e281429898742a2a3bbfe487126916b.png)

```javascript
import { useState, useLayoutEffect } from "react";
import * as ReactDOM from "react-dom";

function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    console.log("=== click ===");
    fetchSomething().then(() => {
      // React 18 使用createRoot，将会批处理这2次更新
      setCount(c => c + 1); // 不会引起重新渲染
      setFlag(f => !f); // 不会引起重新渲染
      // 在最后的一次重新渲染中，更新上面两个状态。
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
      <LogEvents />
    </div>
  );
}

function LogEvents(props) {
  useLayoutEffect(() => {
    console.log("Commit");
  });
  console.log("Render");
  return null;
}

function fetchSomething() {
  return new Promise(resolve => setTimeout(resolve, 100));
}

const rootElement = document.getElementById("root");
ReactDOM.createRoot(rootElement).render(<App />);
```

react18 仅在通常安全的情况下才会进行自动更新。例如react要确保每个用户的启动事件（如单击或者按键），DOM在下一个事件之前完全更新。
**那么， 如果 我不想自动批处理 那怎么办？**
通常，批处理是安全的，但某些代码可能依赖于在状态更改后立即从 DOM 中读取某些内容。对于这些用例，可以使用ReactDOM.flushSync()选择退出自动批处理：

代码如下：

```javascript
import { useState, useLayoutEffect } from "react";
import * as ReactDOM from "react-dom";

function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    console.log("=== click ===");
    fetchSomething().then(() => {
      ReactDOM.flushSync(() => {
        setCount(c => c + 1);
      });
      // 重新渲染一次
      ReactDOM.flushSync(() => {
        setFlag(f => !f);
      });
      // 重新渲染一次
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
      <LogEvents />
    </div>
  );
}

function LogEvents(props) {
  useLayoutEffect(() => {
    console.log("Commit");
  });
  console.log("Render");
  return null;
}

function fetchSomething() {
  return new Promise(resolve => setTimeout(resolve, 100));
}

const rootElement = document.getElementById("root");
ReactDOM.createRoot(rootElement).render(<App />);
```

## Concurrent APIs

并发本身不是一个特性。这是一种新的幕后机制，使 React 能够同时准备多个版本的 UI。您可以将并发视为一个实现细节——它之所以有价值，是因为它解锁了一些特性。React 在其内部实现中使用了复杂的技术，例如优先级队列和多重缓冲 -- 官网

## 1. startTransition

调整渲染优先级。被该函数包裹着的setState触发的渲染被标记为不紧急渲染（其他setState被默认为紧急渲染）；可被紧急渲染抢占进程。

**在startTransiton之前我们先来看看什么是transition?**

react18将更新状态分为两种：紧急更新 && 过渡更新。

紧急更新：反映直接交互的更新，比如打字，点击，按下等。

过渡更新：将UI从一个视图过渡到另一个视图。（如果出现更紧急的更新，该更新会被中断，直至丢弃）

例如：在下拉列表中选择过滤器时，用户希望过滤器在单击之后立即响应。但是，按照程序而言，实际结果可能会存在一个转换过程，比如从开发的角度来说就是先更新过滤的值，再去获取过滤后的数据。这之间的微小延迟通常客户难以察觉到的，客户只关注看到最新的结果。但是为了向后兼容，react18 仍然将更新默认处理为紧急更新，可以通过将更新包装到startTransition，来触发过渡更新。
使用场景：
![在这里插入图片描述](https://img-blog.csdnimg.cn/298aecd69da94167a21ae9b9800b7352.png)

前端做这种数据过滤搜索的话，一般是维护一个state来保存输入搜索框的值，然后去过滤数据列表。但这里会出现一个问题，就是造成输入框输的太快，数据过滤跟不上导致卡顿的问题。在以往的常规操作中我们都会给它抖上一抖。但在react18中我们可以大胆的删除这里的抖动逻辑，然后将第二个更新包装在startTransition.因为从概念来上说，更新输入框的值以及可能会引起一些相关UI的变化，这算是一次紧急更新；第二个搜索结果，相比较于搜索框输入值来说是不太紧急的更新，因为他的结果依赖于完整的input的值。

```javascript
import {transition} from 'react';

const App = () => {
  const [inputValue, setInputValue] = useState();
  const [searchQuery, setSearchQuery] = useState();
  const [filterDate, setFilterData] = useState();

  const onChange = (val) => {
    setInputValue(val); // 紧急更新
  }

  const getQuery= () => {
    startTransition(() => {
      setSearchQuery(inputValue);
    }
  }

  return (
    <div>{filterData}</div>
  )
}


```

更完整使用场景以及实例：https://github.com/reactwg/react-18/discussions/65

**它与setTimeout有什么不同？**

**执行时机上的区别**：transition比setTimeout先执行。

startTransition不会延迟调度，而是立即执行。其接收的函数是同步执行的，只是它内部所有更新都会被标记为'transition'，将一个update的信息传递给了react，由react作为参考信息内部处理执行更新时机。

**是否阻塞页面**：transition可被中断，停止执行。但setTimeout不会。

setTimeout包裹的函数有大量的执行时，仍然会阻塞页面的渲染跟交互，但是transition过渡渲染时，如果同时期存在更紧急的渲染时，该渲染可被中断直至丢弃，所以不会锁定页面。

**附加状态**：setTimeout需要自己添加Loading状态，设置异步状态，但是transition不需要。

## 2. useTransition

react18 为transition提供了一个状态显示，isPending。自动跟踪挂载状态并根据转换状态更新值。当转换挂起时，isPending值为true.

```javascript
import { useTransition } from "react";

const [isPending, startTransition] = useTransition();

JSX: {
  isPending && <Spinner />;
}
```

完整代码实例可参考：https://github.com/reactjs/server-components-demo/blob/629ebda61d085428efd4a5f67f096ca2dfe15a1c/src/SidebarNote.client.js#L53-L59

## 3. useDeferredValue

该hooks可以让我们延迟渲染不紧急的部分（优先渲染紧急部分），类似于防抖但没有固定的时间延迟，实现效果类似于transition，当紧急的任务执行后再得到新的状态值，而这个新的状态值就称作DeferredValue.

与useTransition的不同在于：后者是把transition内部的更新任务变成了过渡更新，而前者是原值通过transition后得到的新的值，把这个新的值作为延时状态。简言之，一个是处理逻辑，一个是生成一个新的状态，

useDeferredValue本质上是在useEffect内部执行的，而useEffect内部是异步执行，所以从运行时机上来看更滞后于useTransition。 (useTransition = useDeferredValue + useEffect)；

```javascript
import { useState, useLayoutEffect, useDeferredValue, memo } from "react";

export default function App() {
  const [value, setValue] = useState("");
  const deferredValue = useDeferredValue(value);

  const handleChange = e => {
    setValue(e.target.value);
  };

  return (
    <div>
      <input value={value} onChange={handleChange} /> // 搜索值
      <LongList value={deferredValue} /> // 展示搜索结果
    </div>
  );
}

const LongList = memo(props => {
  useLayoutEffect(() => {
    // 浏览器渲染前移除大量的 dom 节点，排除浏览器渲染大量节点的影响
    var container = document.getElementsByClassName("container");
    var list = document.getElementsByClassName("list");
    if (list.length) {
      container[0].removeChild(list[0]);
    }
  });

  return (
    <div className="container">
      {Array(100)
        .fill("a")
        .map(item => (
          <div>{props.value}</div>
        ))}
      <div className="list">
        {Array(50000)
          .fill("a")
          .map(item => (
            <div>{props.value}</div>
          ))}
      </div>
    </div>
  );
});
```

完整实例参考：https://codesandbox.io/s/usedeferredvalue-3-demo-forked-qdkvw3?file=/src/App.js

**原理：**
useDeferredValue内部维护了一个useState来更新当前value的值，在useEffect中通过transition模式来调用setState以修改value值，从而返回一个过渡渲染过后新的value值。

内部实现参考代码：

```javascript
function updateDeferredValue(value) {
  const [prevValue, setValue] = updateState(value);
  updateEffect(() => {
    const prevTransition = ReactCurrentBatchConfig.transition;
    ReactCurrentBatchConfig.transition = 1;
    try {
      setValue(value);
    } finally {
      ReactCurrentBatchConfig.transition = prevTransition;
    }
  }, [value]);
  return prevValue;
}
```

更完整原理讲解参考：https://juejin.cn/post/7027995169211285512

## useId

用于在客户端与浏览器之间生成唯一ID值。主要是解决在进行服务端渲染时，如果当前组件已经在服务端渲染过了但是我们客户端并没有什么途径可以知道这个事情，就会重新再渲染一次，从而导致重复渲染的问题。

_先了解一下，服务端渲染（SSR）：_

SSR是指能让你在服务器上将react组件生成html，并将该html发送给用户，SSR能让用户在js包加载和运行之前看到页面的内容。
分为一下几个步骤进行：

1. 在服务器上获取整个应用的数据。
2. 在服务器上将整个应用程序渲染成html并在响应中返回。
3. 在客户端加载整个应用程序的js代码
4. 在客户端将js逻辑绑定到服务器生成的HTML。（hydration）

以上每一步都必须在下一步开始之前完成相应的工作，换言之，其中任何一步延迟都会造成渲染结果缓慢的问题。

**SSR带来的一些问题**

1. 在展示任何东西之前必须获取到所有的东西。
2. 必须优先装好所有的东西，才能在客户端进行hydration。
3. 在有任何交互之前必须hydrate完所有东西。

**\***总结：所有东西 要不全有 要不全无。**\***

## Suspense的SSR支持

react18利用<Suspense>将应用程序分解成小的业务单元，这些小业务单元独立且互不影响。以此来加快展示应用程序，以便用户在等待JS加载时可以看到静态内容，从而提高整体的感知性能。

Suspense主要提供以下两个功能：

**1. 在服务器上流式传输html.**
使用suspense来包裹一些页面部分，使其不需要等待服务端获取数据，react会发送一个占位符（loading状态）开始为页面的其他部分传输HTML，流式HTML的额外内容与`<script>`标签一起放在正确的地方。

**2. 在客户端进行选择性的hydrate**
在HTML和js代码完全下载之前尽早的开始为应用程序进行hydration。他还优先为用户正在互动的部分进行hydration，创造一种即时hydration的错觉。

详细参考文档：https://juejin.cn/post/6982010092258328583

## 总结

1. 添加了useTransition和useDeferredValue, 将紧急更新与转换分开。
2. 添加了useId,用于生成唯一ID。
3. 添加了createRoot，标识使用并发新功能
4. 添加选择性hydrate。
5. 添加了transition为useTransition没有待处理时的callback。
6. strictMode重新运行效果以检查可恢复状态。
7. 添加了useSyncExternalStore以帮助外部存储库与react集成。
8. 添加了useInsertionEffect用于css-in-js库。

**参考文档：**

https://mp.weixin.qq.com/s?__biz=MzkzMjIxNTcyMA==&mid=2247489905&idx=1&sn=16db505dd9f5948bf1639e0fce03db86&chksm=c25e77b6f529fea0d084ce7048e961fbbbb63e4de39405f7744eccc0c74d5cdcfe3b4358ec1c&scene=178&cur_album_id=2099654430097768449#rd
https://github.com/reactwg/react-18/discussions/21
https://github.com/reactwg/react-18/discussions/41
