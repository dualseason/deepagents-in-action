# 第 8 章：多 Agent 协作

> 复杂任务往往需要多个专业 Agent 分工协作。本章用 LangGraph 实现 Supervisor-Worker 模式——一个"主管 Agent"协调多个"专家 Agent"。

## 架构设计

```
用户 → Supervisor（主管）
          │
          ├─→ Researcher（研究员）→ 收集信息
          ├─→ Analyst（分析师）→ 分析数据
          └─→ Writer（写手）→ 撰写报告
          │
          └─→ 输出最终结果
```

Supervisor 根据当前进度决定下一步调用哪个 Agent，直到任务完成。

## Supervisor-Worker 模式实现

```python
from typing import TypedDict, Literal
from langgraph.graph import StateGraph, END
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="deepseek-v4-flash",
    api_key="sk-你的密钥",
    base_url="https://dualseason.com/v1"
)

# 1. 专家 Agent
def researcher(state):
    prompt = f"收集关于以下主题的关键信息: {state['topic']}"
    resp = llm.invoke([HumanMessage(content=prompt)])
    return {"research": resp.content}

def analyst(state):
    prompt = f"基于以下研究结果进行分析: {state['research']}"
    resp = llm.invoke([HumanMessage(content=prompt)])
    return {"analysis": resp.content}

def writer(state):
    prompt = f"撰写报告:\n研究: {state['research']}\n分析: {state['analysis']}"
    resp = llm.invoke([HumanMessage(content=prompt)])
    return {"report": resp.content}

# 2. Supervisor
members = ["researcher", "analyst", "writer"]

system_prompt = (
    "你是一个 Supervisor，负责协调以下 Agent:\n"
    f"{members}\n"
    "根据当前状态决定下一步调用谁，或输出 FINISH 结束。"
)

def supervisor_node(state):
    messages = [
        SystemMessage(content=system_prompt),
        HumanMessage(content=f"当前状态: {state}\n下一步调用谁？")
    ]
    resp = llm.invoke(messages)
    return {"next": resp.content.strip().lower()}

# 3. 状态
class ResearchState(TypedDict):
    topic: str
    research: str
    analysis: str
    report: str
    next: str

# 4. 路由
def router(state) -> Literal["researcher", "analyst", "writer", END]:
    next_agent = state.get("next", "")
    if "finish" in next_agent:
        return END
    for m in members:
        if m in next_agent:
            return m
    return END

# 5. 构建图
workflow = StateGraph(ResearchState)
workflow.add_node("supervisor", supervisor_node)
workflow.add_node("researcher", researcher)
workflow.add_node("analyst", analyst)
workflow.add_node("writer", writer)

workflow.set_entry_point("supervisor")
for m in members:
    workflow.add_edge(m, "supervisor")
workflow.add_conditional_edges("supervisor", router)

app = workflow.compile()

# 6. 执行
result = app.invoke({
    "topic": "AI Agent 的未来发展趋势",
    "research": "", "analysis": "", "report": "", "next": ""
})
print(result["report"])
```

## 设计要点

| 角色 | 职责 | 输入 | 输出 |
|------|------|------|------|
| Supervisor | 协调调度 | 当前状态 | 下一步指令 |
| Worker | 专业执行 | 上游数据 | 处理结果 |

## 其他协作模式

- **广播模式**：所有 Agent 同时收到任务，各自输出后再汇总
- **流水线模式**：Agent 按固定顺序依次处理
- **辩论模式**：多个 Agent 各自提出方案，互相评审

## 小结

多 Agent 协作是构建复杂系统的关键。Supervisor-Worker 模式是最通用的架构，在此基础上可以灵活组合其他模式。
