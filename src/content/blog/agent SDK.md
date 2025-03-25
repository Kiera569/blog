---
author: Kiera
pubDatetime: 2025-03-21
modDatetime: 2025-03-21
title: Agent SDK
slug: AI
featured: false
draft: false
tags:
  - Agent
description: 专为构建 智能代理（AI Agents） 设计的开发工具包，提供了一套标准化接口、预训练模型和功能模块，让开发者能快速组装出能 自主决策、动态交互、多任务协作 的 AI 应用
---

## 基础概念：

专为构建 智能代理（AI Agents） 设计的开发工具包，提供了一套标准化接口、预训练模型和功能模块，让开发者能快速组装出能 自主决策、动态交互、多任务协作 的 AI 应用。

#### 类似粗暴理解：

假设你要造一辆智能汽车，AI Agents包含了方向盘，车轮，传感器，汽车模型等组件，你只需要给他们组合起来，使其能正常行驶即可，无需从造轮胎开始。

## 核心能力：

1.  模块化功能组件

    1. 意图识别：自动解析用户输入的意图
    2. 对话管理：管理多轮对话上下文，避免答非所问
    3. 任务编排：自动分解复杂任务 （调用API或者其他外部工具）
    4. 记忆与学习：记录历史交互，逐步优化决策。

2.  无缝集成能力

    1. 连接大模型：支持对接GPT，claude等大模型，处理复杂推理。
    2. 接入外部工具：调用数据库，API,硬件设备等
    3. 多模态交互：支持文本，语音，图像的输入输出。

3.  开箱即用的基础设施
    1. 开发调试工具：可视化对话流设计器，实时日志监控；
    2. 部署托管：一键部署到云服务器或者边缘设备；
    3. 安全合规：内置数据加密，权限控制和审计日志等。

## 典型的应用场景：

1.  智能客服；
2.  自动化数据分析；
3.  家庭智能管家；

## Agent SDK 选择：

1.  功能完整度
2.  扩展性
3.  生态支持
4.  成本：开源免费vs按调用量付费

## 如何从0-1利用Langchain Agents, 实现一个天气助手：

#### 技术栈：Python + Langchain + OpenWeatherMap api + OpenAi GPT

#### 环境准备

1. 安装python pip.
   如果安装了python 但没有pip命令：

```
// 安装pip
python -m ensurepip --upgrade --user

// 查看安装地址
python -m site --user-base

// 配置环境变量， 将安装地址写入到.zshrc文件中
vim ~/.zshrc

export PATH="$PATH:/home/your_username/.local/bin"

```

2. 安装langchain, openai, python-dotenv, request等依赖项。

```
# weather_agent_v1.py
from langchain.agents import Tool, initialize_agent
from langchain.utilities import OpenWeatherMapAPIWrapper
from langchain.chat_models import ChatOpenAI
from dotenv import load_dotenv
import os

# 加载环境变量
load_dotenv()

# 1. 初始化天气工具
weather = OpenWeatherMapAPIWrapper()
weather_tool = Tool(
    name="Weather",
    func=weather.run,
    description="查询城市天气，输入格式：'城市名,国家码'（如'London,GB'）"
)

# 2. 初始化语言模型
llm = ChatOpenAI(
    temperature=0,  # 控制输出随机性（0为最确定）
    openai_api_key=os.getenv("OPENAI_API_KEY")
)

# 3. 创建Agent
agent = initialize_agent(
    tools=[weather_tool],
    llm=llm,
    agent="zero-shot-react-description",  # 简单任务适用
    verbose=True  # 打印执行细节（调试用）
)

# 4. 测试查询
response = agent.run("成都现在的温度是多少？")
print(response)
```
