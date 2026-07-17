# 第 2 章：LCEL 链式调用

> LangChain Expression Language（LCEL）是 LangChain 的声明式编程范式。通过 `|` 管道符将组件串联起来，构建清晰的数据流。

## 从 Prompt 到输出的流水线

LCEL 的核心思想是"管道"——上一个组件的输出自动成为下一个组件的输入：

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

llm = ChatOpenAI(
    model="deepseek-v4-flash",
    api_key="sk-你的密钥",
    base_url="https://onekey.dualseason.com/v1"
)

prompt = ChatPromptTemplate.from_template(
    "用{language}写一个{task}函数"
)

chain = prompt | llm | StrOutputParser()

result = chain.invoke({
    "language": "Python",
    "task": "快速排序"
})
print(result)
```

这个简单的链做了三件事：

1. `prompt`：将模板和变量组合成完整的提示
2. `llm`：将提示发送给模型
3. `StrOutputParser`：从模型响应中提取文本

## 并行调用

LCEL 的 `RunnableParallel` 可以同时执行多个分支：

```python
from langchain_core.runnables import RunnableParallel

prompt1 = ChatPromptTemplate.from_template("翻译成英文: {text}")
prompt2 = ChatPromptTemplate.from_template("翻译成日文: {text}")

chain1 = prompt1 | llm | StrOutputParser()
chain2 = prompt2 | llm | StrOutputParser()

parallel = RunnableParallel(en=chain1, ja=chain2)

result = parallel.invoke({"text": "人工智能正在改变世界"})
print(f"英文: {result['en']}")
print(f"日文: {result['ja']}")
```

## Chain 组合

你可以将多个 Chain 组合成一个更大的 Chain——一个 Chain 的输出作为下一个 Chain 的输入：

```python
prompt_en = ChatPromptTemplate.from_template("翻译成英文: {topic}")
prompt_funny = ChatPromptTemplate.from_template("用幽默的方式解释: {text}")

chain = (
    {"text": prompt_en | llm | StrOutputParser()}
    | prompt_funny
    | llm
    | StrOutputParser()
)

result = chain.invoke({"topic": "机器学习"})
print(result)
```

这里先将 `topic` 翻译成英文，再把英文结果用幽默方式解释。

## 小结

LCEL 是 LangChain 的基石。理解 `|` 管道和 `RunnableParallel` 后，你已经掌握了构建复杂 LLM 应用的基本方法。下一章我们在此基础上添加工具调用能力。
