# 第 9 章：人工审批流程

> Human-in-the-Loop（HITL）是在 Agent 执行流程中插入人工审批节点。对于敏感操作（如发送邮件、执行命令），需要人来确认。

## 为什么需要 HITL？

Agent 再智能也会有犯错的时候。在以下场景中，人工审批是必要的：

- **财务操作**：支付、退款、转账
- **内容发布**：自动生成的文章、评论
- **数据删除**：删除用户数据或数据库记录
- **权限变更**：修改系统配置或用户权限

## LangGraph 的中断机制

LangGraph 的 `interrupt_before` 可以在进入指定节点前暂停执行，等待外部输入：

```python
from typing import TypedDict
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.memory import MemorySaver
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="deepseek-v4-flash",
    api_key="sk-你的密钥",
    base_url="https://onekey.dualseason.com/v1"
)

class ApprovalState(TypedDict):
    query: str
    plan: str
    approved: bool
    result: str

def planner(state: ApprovalState) -> dict:
    resp = llm.invoke([
        HumanMessage(content=f"为以下任务制定执行计划: {state['query']}")
    ])
    return {"plan": resp.content}

def executor(state: ApprovalState) -> dict:
    if not state.get("approved"):
        return {"result": "未审批，不执行"}
    resp = llm.invoke([
        HumanMessage(content=f"执行计划: {state['plan']}")
    ])
    return {"result": resp.content}

# 构建图 —— 在 executor 前中断
workflow = StateGraph(ApprovalState)
workflow.add_node("planner", planner)
workflow.add_node("executor", executor)
workflow.set_entry_point("planner")
workflow.add_edge("planner", "executor")
workflow.add_edge("executor", END)

app = workflow.compile(
    interrupt_before=["executor"],
    checkpointer=MemorySaver()
)

# 第一轮：生成计划（会停在 executor 前）
result = app.invoke(
    {"query": "给所有用户发送系统升级通知", "plan": "", "approved": False, "result": ""},
    config={"configurable": {"thread_id": "approval-1"}}
)

print(f"计划: {result['plan']}")
print(f"当前节点: {app.get_state({'configurable': {'thread_id': 'approval-1'}}).next}")

# 模拟人工审批
approved = True  # 实际中需要用户确认

# 第二轮：继续执行（传入审批结果）
result = app.invoke(
    {"approved": approved},
    config={"configurable": {"thread_id": "approval-1"}}
)

print(f"结果: {result['result']}")
```

## 审批流程可视化

```
用户输入 → Planner（制定计划）→ [暂停] → 人工审核
                                        ↓ 通过
                                    Executor（执行）
                                        ↓
                                    输出结果
```

## 风险等级策略

| 风险等级 | 操作类型 | 审批方式 |
|----------|----------|----------|
| 低 | 搜索、阅读 | 自动放行 |
| 中 | 写入文件、回复用户 | 记录日志 |
| 高 | 删除数据、支付 | 必须人工确认 |
| 严重 | 系统配置变更 | 多人审批 |

## 小结

HITL 是 Agent 从"玩具"走向"生产"的关键能力。通过 `interrupt_before` 和 checkpointer，LangGraph 提供了优雅的暂停-审批-恢复机制。
