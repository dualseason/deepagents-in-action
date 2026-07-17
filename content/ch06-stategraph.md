# 第 6 章：StateGraph 状态图

> LangGraph 的核心是 StateGraph——一个有状态、带条件分支的图执行引擎。与 LCEL 的固定流水线不同，StateGraph 支持循环和复杂路由。

## 为什么需要状态图？

LCEL 链是直线流水线——数据从 A 到 B 到 C，一路到底。

但 Agent 的工作流不是线性的——它可能需要循环（思考→行动→观察→再思考）、条件分支（是调用工具还是直接回答）、并行执行等。

StateGraph 就是为了解决这个问题而生的。

## 核心概念

```
State（状态）→ Node（节点）→ Edge（边）→ Conditional Edge（条件边）
```

- **State**：应用状态，所有节点共享读写
- **Node**：处理节点，接收 State 返回更新
- **Edge**：固定连接，A 执行完后到 B
- **Conditional Edge**：条件路由，根据 State 动态决定下一步

## 构建你的第一个图形

```python
from typing import TypedDict, Literal
from langgraph.graph import StateGraph, END
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage

llm = ChatOpenAI(
    model="deepseek-v4-flash",
    api_key="sk-你的密钥",
    base_url="https://dualseason.com/v1"
)

# 1. 定义状态
class AgentState(TypedDict):
    input: str
    reasoning: str
    answer: str
    steps: int

# 2. 定义节点
def reason_node(state: AgentState) -> dict:
    prompt = f"分析以下问题，给出解决思路: {state['input']}"
    resp = llm.invoke([HumanMessage(content=prompt)])
    return {"reasoning": resp.content, "steps": state["steps"] + 1}

def answer_node(state: AgentState) -> dict:
    prompt = f"问题: {state['input']}\n分析: {state['reasoning']}\n请给出最终答案:"
    resp = llm.invoke([HumanMessage(content=prompt)])
    return {"answer": resp.content, "steps": state["steps"] + 1}

# 3. 条件判断
def should_continue(state: AgentState) -> Literal["answer", "reason"]:
    if state["steps"] < 1:
        return "reason"  # 继续分析
    return "answer"      # 给出答案

# 4. 构建图
workflow = StateGraph(AgentState)
workflow.add_node("reason", reason_node)
workflow.add_node("answer", answer_node)
workflow.set_entry_point("reason")
workflow.add_conditional_edges("reason", should_continue)
workflow.add_edge("answer", END)

app = workflow.compile()

# 5. 运行
result = app.invoke({
    "input": "解释量子计算的基本原理",
    "reasoning": "",
    "answer": "",
    "steps": 0
})

print(f"推理: {result['reasoning']}")
print(f"答案: {result['answer']}")
```

## 与 LCEL 的区别

| 特性 | LCEL | StateGraph |
|------|------|------------|
| 数据流 | 直线管道 | 图结构 |
| 状态管理 | 无（每次重新计算） | 共享状态 |
| 循环 | 不支持 | 原生支持 |
| 条件分支 | 不支持 | Conditional Edge |
| 适用场景 | 简单流水线 | Agent 循环、复杂编排 |

## 小结

StateGraph 是构建 Agent 的基础。有了状态管理和条件路由，我们可以实现真正的 ReAct 循环。下一章将用 StateGraph 实现一个完整的 ReAct Agent。
