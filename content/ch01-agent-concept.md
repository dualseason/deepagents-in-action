# 第 1 章：什么是 AI Agent

> AI Agent（智能体）是能够自主感知环境、做出决策并执行行动的智能程序。与传统的 LLM 调用不同，Agent 拥有"思考-行动-观察"的循环能力。

## Agent 的核心三要素

一个完整的 AI Agent 由三部分组成：

### 1. 大脑（LLM）

大语言模型作为 Agent 的"大脑"，负责推理、规划和决策。它接收信息、分析问题、决定下一步行动。

### 2. 工具（Tools）

工具是 Agent 与外部世界交互的接口，包括：

- 搜索引擎（获取实时信息）
- 计算器（数学运算）
- 文件操作（读写文件）
- API 调用（访问外部服务）
- 数据库查询

### 3. 记忆（Memory）

记忆让 Agent 拥有上下文感知能力：

- 短期记忆：当前对话窗口内的信息
- 长期记忆：跨会话持久化存储

## ReAct 模式

ReAct（Reasoning + Acting）是 Agent 最核心的工作模式：

```
Thought（思考）→ Action（行动）→ Observation（观察）→ Thought（再思考）→ ...
```

每一步 LLM 先思考当前状态，决定调用哪个工具，然后观察工具返回结果，再继续推理，直到得出最终答案。

## 最小 Agent 实现

用几十行代码实现一个最简 Agent：

```python
from openai import OpenAI
import json

client = OpenAI(
    api_key="sk-你的密钥",
    base_url="https://onekey.dualseason.com/v1"
)

# 定义工具
tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "查询城市天气",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {"type": "string", "description": "城市名称"}
            },
            "required": ["city"]
        }
    }
}]

def get_weather(city):
    data = {"北京": "25°C 晴", "上海": "28°C 多云"}
    return data.get(city, "数据不可用")

messages = [{"role": "user", "content": "北京今天天气怎么样？"}]

resp = client.chat.completions.create(
    model="deepseek-v4-flash",
    messages=messages,
    tools=tools
)

msg = resp.choices[0].message
if msg.tool_calls:
    for tc in msg.tool_calls:
        args = json.loads(tc.function.arguments)
        result = get_weather(args["city"])
        print(f"天气结果: {result}")
```

这个例子展示了 Agent 最核心的能力：LLM 自主判断需要调用工具，解析参数，我们执行工具并返回结果。

## 为什么需要框架？

手写 Agent 虽然能理解原理，但在实际项目中会遇到：

- 复杂的工具编排逻辑
- 多轮对话状态管理
- 多个 Agent 之间的协作
- 错误处理和重试
- 可观测性和调试

这就是后续章节中 LangChain 和 LangGraph 要解决的问题。
