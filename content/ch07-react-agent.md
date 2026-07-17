# 第 7 章：ReAct Agent 实战

> 本章用 LangGraph 的 StateGraph 实现一个完整的 ReAct Agent——它能自主决定调用工具还是直接回答，并在循环中不断推理。

## Agent 循环架构

```
用户输入 → Agent(LLM) → 有工具调用？→ 是 → ToolNode → Agent(LLM)
                          ↓ 否                      ↑
                          └──→ 输出最终答案 ─────────┘
```

这个循环用 StateGraph 实现非常自然：Agent 节点和 ToolNode 之间形成环路。

## 完整实现

```python
from typing import TypedDict, Literal
from langgraph.graph import StateGraph, END
from langgraph.prebuilt import ToolNode
from langchain_core.messages import HumanMessage
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="deepseek-v4-flash",
    api_key="sk-你的密钥",
    base_url="https://onekey.dualseason.com/v1"
)

# 1. 定义工具
@tool
def get_weather(city: str) -> str:
    """查询城市天气"""
    data = {"北京": "25°C 晴", "上海": "28°C 多云", "纽约": "22°C 阴"}
    return data.get(city, f"{city} 数据不可用")

@tool
def get_time(city: str) -> str:
    """查询城市当前时间"""
    data = {"北京": "10:30", "上海": "10:30", "纽约": "22:30"}
    return data.get(city, f"{city} 时间数据不可用")

tools = [get_weather, get_time]
llm_with_tools = llm.bind_tools(tools)
tool_node = ToolNode(tools)

# 2. 定义状态
class AgentState(TypedDict):
    messages: list

# 3. 定义节点
def call_model(state: AgentState) -> dict:
    response = llm_with_tools.invoke(state["messages"])
    return {"messages": [response]}

def should_continue(state: AgentState) -> Literal["tools", END]:
    last = state["messages"][-1]
    if hasattr(last, "tool_calls") and last.tool_calls:
        return "tools"
    return END

# 4. 构建图
workflow = StateGraph(AgentState)
workflow.add_node("agent", call_model)
workflow.add_node("tools", tool_node)
workflow.set_entry_point("agent")
workflow.add_conditional_edges("agent", should_continue)
workflow.add_edge("tools", "agent")

app = workflow.compile()

# 5. 运行
result = app.invoke({
    "messages": [HumanMessage(content="北京天气怎么样？纽约现在几点？")]
})

print(result["messages"][-1].content)
```

## 关键点

1. **`bind_tools`**：将工具定义注入 LLM，使其知道有哪些工具可用
2. **`ToolNode`**：自动解析 `tool_calls` 并执行对应的工具
3. **条件边**：判断 LLM 的输出是工具调用还是最终回答
4. **循环**：工具执行后回到 Agent 节点，形成 ReAct 循环

## 调试技巧

LangGraph 提供了丰富的调试能力：

```python
# 查看执行步骤
for event in app.stream({"messages": [HumanMessage(content="北京天气？")]}):
    for node, value in event.items():
        print(f"节点 {node}: {value}")
```

## 小结

现在你已经可以构建一个完整的 Agent——它能自主使用工具、循环推理、最终回答问题。下一章我们将在这个基础上让多个 Agent 协作。
