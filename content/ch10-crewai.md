# 第 10 章：CrewAI 多角色协作框架

> CrewAI 是更高层的多 Agent 框架，通过"角色扮演"的方式组织多个 Agent 协作。每个 Agent 有角色、目标和背景故事。

## 核心概念

```
Agent（角色）→ Task（任务）→ Crew（团队）→ Kickoff（执行）
```

- **Agent**：扮演某个角色，有特定目标和能力
- **Task**：分配给 Agent 的任务，有描述和期望输出
- **Crew**：管理多个 Agent 和 Task 的执行流程

## 快速上手

```python
from crewai import Agent, Task, Crew
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="deepseek-v4-flash",
    api_key="sk-你的密钥",
    base_url="https://onekey.dualseason.com/v1"
)

# 1. 定义角色
researcher = Agent(
    role="高级研究员",
    goal="收集和分析信息",
    backstory="你经验丰富，擅长多角度分析问题",
    llm=llm,
    verbose=True
)

writer = Agent(
    role="技术作者",
    goal="撰写清晰易懂的技术文章",
    backstory="你是优秀的技术写手，能把复杂概念讲得通俗易懂",
    llm=llm,
    verbose=True
)

# 2. 定义任务
research_task = Task(
    description="研究 AI Agent 的核心概念：ReAct 模式、工具调用和记忆机制",
    expected_output="一份详细的研究报告",
    agent=researcher
)

write_task = Task(
    description="基于研究报告写一篇面向初学者的介绍文章",
    expected_output="一篇通俗易懂的技术文章",
    agent=writer
)

# 3. 组建团队并执行
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    verbose=True
)

result = crew.kickoff()
print(result)
```

## 与 LangGraph 比较

| 特性 | LangGraph | CrewAI |
|------|-----------|--------|
| 抽象层次 | 底层图引擎 | 高层角色框架 |
| 控制粒度 | 精细（节点/边/状态） | 粗粒度（角色/任务） |
| 学习曲线 | 较陡 | 平缓 |
| 适用场景 | 需要细粒度控制 | 快速构建多Agent应用 |
| 灵活性 | 高 | 中等 |

## 给 Agent 绑定工具

```python
from langchain_core.tools import tool

@tool
def search_kb(query: str) -> str:
    """搜索知识库"""
    data = {"langchain": "LangChain 是构建 LLM 应用的框架"}
    return data.get(query.lower(), f"未找到 '{query}'")

researcher_with_tools = Agent(
    role="研究员",
    goal="搜索知识库提供信息",
    backstory="你精通知识库检索",
    tools=[search_kb],
    llm=llm
)
```

## 小结

CrewAI 适合快速构建多角色协作场景。当你需要的是"研究员 + 写手 + 审核"这样角色分明的团队时，CrewAI 比手写 LangGraph 更高效。
