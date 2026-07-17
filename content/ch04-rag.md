# 第 4 章：RAG 检索增强生成

> RAG（Retrieval Augmented Generation）让 LLM 能够访问外部知识库，基于检索到的信息回答问题，有效解决知识截止和幻觉问题。

## RAG 流程

RAG 的标准流程分为五步：

1. **文档加载**：读取 PDF、网页、数据库等来源
2. **文档分割**：将长文档切分成适合检索的块
3. **向量化存储**：用 Embedding 模型将文本转为向量，存入向量库
4. **检索**：根据用户问题检索最相关的文档片段
5. **增强生成**：将检索到的文档作为上下文，让 LLM 生成回答

## 实现一个 RAG 系统

```python
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_chroma import Chroma

llm = ChatOpenAI(
    model="deepseek-v4-flash",
    api_key="sk-你的密钥",
    base_url="https://onekey.dualseason.com/v1"
)

embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small",
    api_key="sk-你的密钥",
    base_url="https://onekey.dualseason.com/v1"
)

# 1. 准备知识库
docs = [
    "LangChain 是一个 LLM 应用开发框架，支持链式调用和工具集成。",
    "LangGraph 在 LangChain 基础上增加了状态图编排能力。",
    "ReAct 模式让 LLM 交替进行推理和行动，是 Agent 的核心范式。",
]

# 2. 分割并存入向量库
text_splitter = RecursiveCharacterTextSplitter(chunk_size=100, chunk_overlap=20)
chunks = text_splitter.create_documents(docs)

vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    collection_name="agent_kb"
)

retriever = vectorstore.as_retriever(search_kwargs={"k": 2})

# 3. 构建 RAG 链
template = """基于以下上下文回答问题：

{context}

问题: {question}"""

prompt = ChatPromptTemplate.from_template(template)

def format_docs(docs):
    return "\n---\n".join(d.page_content for d in docs)

rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

# 4. 测试
result = rag_chain.invoke("LangGraph 和 LangChain 有什么关系？")
print(result)
```

## 检索策略

| 策略 | 说明 | 适用场景 |
|------|------|----------|
| 相似度检索 | 按向量距离排序 | 通用场景 |
| MMR | 兼顾相关性和多样性 | 避免重复内容 |
| 混合检索 | 向量 + 关键词 | 需要精确匹配 |

## 小结

RAG 是构建知识密集型 Agent 的基础能力。结合前一章的工具调用，Agent 既可以检索知识库，又可以调用外部工具，两种能力互补。
