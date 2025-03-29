---
author: Kiera
pubDatetime: 2025-03-26
modDatetime: 2025-03-26
title: 初探MCP和SSE
slug: mcp
featured: true
draft: false
tags:
  - ai
description: Model Context Protocol，模型上下文协议；Server Sent Event, 服务器发送事件。
---

# MCP

官方文档：https://modelcontextprotocol.io/introduction

## 什么是MCP？

一种开放协议，它标准化了应用程序如何为LLM提供上下文。正如USB接口一样，它也提供了一种AI模型连接到不同数据源和工具的标准化方式。

## MCP有什么优点？

协助在LLM上构建代理和更复杂的应用流程。

1. 使LLM可以直接插入预构建集成；

2. 在不同的应用/服务之间灵活切换切保持上下文，从而增强整体自主执行任务的能力；

3. 更好的提供数据保护。

## MCP组成

1. MCP Host: MCP启动连接的程序；

2. MCP Client: 在HOST程序内维护与服务端的1:1连接；

3. MCP Server: 通过标准化协议，为client提供上下文，工具和提示。

4. Local Data Source: 本地数据库；

5. Remote Services: 外部文件，数据库或者API

# SSE（server-sent-event, 服务器发送事件）

MDN: https://developer.mozilla.org/zh-CN/docs/Web/API/Server-sent_events/Using_server-sent_events

## 什么是SSE

服务器发送事件，是一种基于HTTP的实时传输技术，允许服务器向客户端主动推送数据流。

## 核心特性

1. 单向通信；

   - 单向数据流， 服务器向客户端推送；

   - 无需客户端轮询 获取数据；

2. 基于HTTP；

   - 无需其他任何协议（如web-socket）

   - 支持跨域（配置CORS）

3. 自动重连；

   - 断开后浏览器尝试自动连接。"retry"

4. 轻量高效

   - 数据以纯文本格式传输，减小开销。

## 工作原理

1. 客户端发起连接请求（浏览器对同一域名下的 SSE 连接数有限制（通常为 6 个），超出后需排队。）

2. 服务端响应请求，保持长连接

3. 服务器推送数据（每条消息必须以 data: 开头，以 \n\n 结尾，支持多行数据和自定义事件类型）

4. 客户端接收数据并处理

```js
// 客户端
// 通过eventSource发起请求
const eventSource = new EventSource("/sse-endpoint");
// 监听数据
eventSource.onmessage = event => {
  document.getElementById("output").innerHTML += event.data + "<br>";
};

// 监听错误
eventSource.onerror = err => {
  console.error("SSE 连接错误:", err);
};

//  关闭事件流
eventSource.close();

// 服务端
const express = require("express");
const app = express();

app.get("/sse-endpoint", (req, res) => {
  // 设置Content-Type，设置保持长连接
  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.setHeader("Connection", "keep-alive");

  // 每隔1秒发送数据
  const interval = setInterval(() => {
    res.write(`data: 当前时间 - ${new Date().toISOString()}\n\n`);
  }, 1000);

  // 客户端断开连接时清理
  req.on("close", () => {
    clearInterval(interval);
    res.end();
  });
});

app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

## 适用场景

1. 实时通知：新消息提醒、订单状态更新、系统报警等

2. 数据监控：实时展示服务器 CPU 使用率、在线用户数等

3. 动态内容更新：新闻推送、股票价格变动、体育赛事比分。

4. 日志流：实时输出应用日志，方便开发者调试。
