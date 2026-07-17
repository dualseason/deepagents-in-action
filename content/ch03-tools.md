# 第 3 章：工具定义与调用

> 工具（Tools）是 Agent 与外部世界交互的桥梁。LangChain 提供了简洁的 `@tool` 装饰器和标准的工具调用接口。

## 定义工具

使用 `@tool` 装饰器可以快速将函数变成工具：

```python
from langchain_core.tools import tool

@tool
def get_weather(city: str) -> str:
    """查询指定城市的天气"""
    data = {"北京": "25°C 晴", "上海": "28°C 多云"}
    return data.get(city, f"{city} 天气数据不可用")

@tool
def multiply(a: int, b: int) -> int:
    """计算两个数的乘积"""
    return a * b

print(f"工具名: {get_weather.name}")   # get_weather
print(f"描述: {get_weather.description}")  # 查询指定城市的天气
print(f"参数: {get_weather.args}")      # {'city': ...}
```

函数的类型注解和文档字符串会自动提取为工具的 JSON Schema，供 LLM 识别。

## 绑定工具到 LLM

将工具绑定到 LLM 后，模型就能在需要时自动调用它们：

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="deepseek-v4-flash",
    api_key="sk-你的密钥",
    base_url="https://dualseason.com/v1"
)

tools = [get_weather, multiply]
llm_with_tools = llm.bind_tools(tools)

resp = llm_with_tools.invoke("北京天气怎么样？再算 6*8")

for tc in resp.tool_calls:
    print(f"工具: {tc['name']}")
    print(f"参数: {tc['args']}")

    if tc["name"] == "get_weather":
        print(get_weather.invoke(tc["args"]))
    elif tc["name"] == "multiply":
        print(multiply.invoke(tc["args"]))
```

`bind_tools` 自动将工具的定义以 LLM 所能理解的格式传入。当 LLM 判断需要调用工具时，`tool_calls` 字段会包含工具名称和参数。

## 工具调用的完整流程

```
用户输入 → LLM分析 → 决定调用工具 → 提取参数
    ↑                              ↓
    └────────── 返回结果 ←──────────┘
```

LangChain 的 `ToolNode` 可以帮助自动化这个流程：

```python
from langgraph.prebuilt import ToolNode

tool_node = ToolNode([get_weather, multiply])
# 自动解析 LLM 的 tool_calls 并执行对应工具
```

## 小结

工具是 Agent 能力的延伸。通过 `@tool` 装饰器和 `bind_tools`，可以轻松地为 LLM 添加各种能力——搜索、计算、文件操作、API 调用等。在后续章节中，工具将与 RAG 和 Agent 循环深度结合。
