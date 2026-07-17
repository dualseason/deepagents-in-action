<div align="center">

# AI Agent 实战教程

**从零开始，系统学习 AI Agent 开发 — LangChain / LangGraph / 上层框架 一网打尽**

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/协议-CC%20BY--NC--SA%204.0-lightgrey)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)](CONTRIBUTING.md)

<br/>

所有示例通过 **[双季中转站](https://dualseason.com)** 调用 LLM，默认模型 **deepseek-v4-flash**。

</div>

## 课程大纲

### 准备篇

| 章节 | 内容 |
|------|------|
| 环境配置 | Python 环境搭建、中转站 API 配置、依赖安装 |

### 认知篇

| 章节 | 内容 |
|------|------|
| 第 1 章 | [什么是 AI Agent — 核心概念与最小实现](https://dualseason.github.io/deepagents-in-action/chapters/ch01-agent-concept/) |

### 核心篇（LangChain）

| 章节 | 内容 |
|------|------|
| 第 2 章 | [LCEL 链式调用 — LangChain 声明式编程](https://dualseason.github.io/deepagents-in-action/chapters/ch02-lcel/) |
| 第 3 章 | [工具定义与调用 — Agent 的能力延伸](https://dualseason.github.io/deepagents-in-action/chapters/ch03-tools/) |
| 第 4 章 | [RAG 检索增强生成 — 让 Agent 拥有知识库](https://dualseason.github.io/deepagents-in-action/chapters/ch04-rag/) |
| 第 5 章 | [记忆机制 — 让 Agent 拥有上下文感知](https://dualseason.github.io/deepagents-in-action/chapters/ch05-memory/) |

### 进阶篇（LangGraph）

| 章节 | 内容 |
|------|------|
| 第 6 章 | [StateGraph 状态图 — LangGraph 核心引擎](https://dualseason.github.io/deepagents-in-action/chapters/ch06-stategraph/) |
| 第 7 章 | [ReAct Agent 实战 — LangGraph 实现 Agent 循环](https://dualseason.github.io/deepagents-in-action/chapters/ch07-react-agent/) |
| 第 8 章 | [多 Agent 协作 — Supervisor-Worker 模式](https://dualseason.github.io/deepagents-in-action/chapters/ch08-multi-agent/) |
| 第 9 章 | [人工审批流程 — Human-in-the-Loop](https://dualseason.github.io/deepagents-in-action/chapters/ch09-human-in-the-loop/) |

### 实战篇

| 章节 | 内容 |
|------|------|
| 第 10 章 | [CrewAI 入门 — 多角色协作框架](https://dualseason.github.io/deepagents-in-action/chapters/ch10-crewai/) |
| 第 11 章 | [实战项目：智能客服系统](https://dualseason.github.io/deepagents-in-action/chapters/ch11-customer-service/) |

## 技术栈

- **LLM 中转站**：[双季中转站](https://dualseason.com)（兼容 OpenAI API）
- **默认模型**：deepseek-v4-flash
- **框架**：LangChain / LangGraph / CrewAI
- **前端**：[Astro 6](https://astro.build/) + [Tailwind CSS 4](https://tailwindcss.com/)

## 本地开发

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build
```

## 项目结构

```
content/               # 课程内容（Markdown）
scripts/chapters.json  # 章节元数据
src/                   # 前端组件和页面
```

## 协议

课程内容采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh) 协议。
网站源码采用 [MIT](https://opensource.org/license/mit) 协议。
