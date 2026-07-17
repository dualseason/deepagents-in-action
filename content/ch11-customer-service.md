# 第 11 章：智能客服系统实战

> 综合运用前 10 章所学，构建一个完整的智能客服系统：意图识别 + 多专家 Agent + RAG 知识库。

## 系统架构

```
用户输入 → 意图路由器
                │
        ┌───────┼───────┐
        ▼       ▼       ▼
    产品专家  技术专家  售后专家
        │       │       │
    ┌───┘       │       └───┐
    ▼           ▼           ▼
  产品知识库  技术知识库  售后知识库
                │
                ▼
            输出回答
```

## 第一步：构建知识库

```python
from langchain_openai import OpenAIEmbeddings
from langchain_chroma import Chroma

embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small",
    api_key="sk-你的密钥",
    base_url="https://dualseason.com/v1"
)

# 产品知识
product_docs = [
    "基础版 $99/月，专业版 $299/月，企业版定制报价",
    "支持 14 天免费试用，无需信用卡",
    "支持 LangChain、LangGraph、CrewAI 集成",
]

product_kb = Chroma.from_texts(
    texts=product_docs,
    embedding=embeddings,
    collection_name="product_kb"
)

# 技术知识
tech_docs = [
    "API 密钥在控制台 -> 设置 -> API Keys 生成",
    "支持 Python、JavaScript、Java SDK",
    "速率限制：免费版 100次/分钟，专业版 1000次/分钟",
]

tech_kb = Chroma.from_texts(
    texts=tech_docs,
    embedding=embeddings,
    collection_name="tech_kb"
)
```

## 第二步：构建客服 Agent

```python
from typing import TypedDict, Literal
from langgraph.graph import StateGraph, END
from langchain_core.messages import HumanMessage

class State(TypedDict):
    input: str
    intent: str
    context: str
    response: str

# 意图识别
def router_node(state: State) -> dict:
    prompt = f"""判断意图，返回一个词：
- product: 产品咨询
- tech: 技术问题
- support: 售后服务
- chat: 其他

问题: {state['input']}"""
    resp = llm.invoke([HumanMessage(content=prompt)])
    return {"intent": resp.content.strip().lower()}

# 专家节点
def product_expert(state: State) -> dict:
    docs = product_kb.similarity_search(state["input"], k=2)
    context = "\n".join(d.page_content for d in docs)
    prompt = f"知识库:\n{context}\n\n问题: {state['input']}"
    resp = llm.invoke([HumanMessage(content=prompt)])
    return {"context": context, "response": resp.content}

def tech_expert(state: State) -> dict:
    docs = tech_kb.similarity_search(state["input"], k=2)
    context = "\n".join(d.page_content for d in docs)
    prompt = f"知识库:\n{context}\n\n问题: {state['input']}"
    resp = llm.invoke([HumanMessage(content=prompt)])
    return {"context": context, "response": resp.content}

# 路由
def route_intent(state) -> Literal["product", "tech", "support", "chat"]:
    return state.get("intent", "chat")

# 构建图
workflow = StateGraph(State)
workflow.add_node("router", router_node)
workflow.add_node("product", product_expert)
workflow.add_node("tech", tech_expert)

workflow.set_entry_point("router")
workflow.add_conditional_edges("router", route_intent, {
    "product": "product",
    "tech": "tech",
})
for node in ["product", "tech"]:
    workflow.add_edge(node, END)

app = workflow.compile()
```

## 运行效果

```python
result = app.invoke({
    "input": "你们专业版多少钱？",
    "intent": "",
    "context": "",
    "response": ""
})
print(f"意图: {result['intent']}")
print(f"回答: {result['response']}")
```

## 扩展方向

- **增加更多专家**：添加物流查询、退换货处理等
- **人工转接**：当 Agent 无法回答时转接人工客服
- **对话历史**：集成记忆模块，保持上下文
- **多轮对话**：支持追问和澄清

## 小结

本章整合了 RAG（知识库）、LangGraph（状态图路由）、工具调用等核心能力，构建了一个可扩展的客服系统。这套架构可以复用到几乎所有需要"分类 + 专业处理"的场景。
