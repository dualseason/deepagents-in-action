# 第 5 章：记忆机制

> 记忆让 Agent 拥有跨对话的上下文感知能力。没有记忆的 Agent 每次对话都是"第一次见面"。

## 三种记忆层次

### 1. 对话缓冲记忆（ConversationBufferMemory）

最简单的记忆形式——完整保留所有历史消息：

```python
from langchain.memory import ChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个 AI 助手。"),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}")
])

chain = prompt | llm

store = {}

def get_session_history(session_id: str):
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]

chain_with_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="history"
)

resp = chain_with_history.invoke(
    {"input": "我叫小明"},
    config={"configurable": {"session_id": "user-1"}}
)

resp = chain_with_history.invoke(
    {"input": "你还记得我叫什么吗？"},
    config={"configurable": {"session_id": "user-1"}}
)
print(resp.content)  # 记得！你叫小明
```

### 2. 总结记忆（ConversationSummaryMemory）

当对话很长时，缓冲记忆会消耗大量 token。总结记忆用 LLM 定期总结对话内容。

### 3. 向量检索记忆（VectorStoreRetrieverMemory）

将历史对话向量化存储，只检索与当前问题最相关的记忆。适合长期、大规模的对话历史。

## 在 Agent 中集成记忆

```python
from langchain.memory import ConversationBufferMemory
from langchain.agents import AgentExecutor, create_react_agent

memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    memory=memory,
    verbose=True
)
```

## 记忆策略选择

| 场景 | 推荐策略 |
|------|----------|
| 简短问答 | 缓冲记忆 |
| 长对话 | 总结记忆 |
| 长期用户画像 | 向量检索记忆 |
| 复杂 Agent | 组合使用多种记忆 |

## 小结

记忆是 Agent 从"工具"进化到"助手"的关键。选择合适的记忆策略，能让 Agent 在长对话中保持一致的行为和个性化响应。
