# 准备篇：环境配置

> 工欲善其事，必先利其器。本章带领你完成课程所需的环境搭建。

## 前提条件

- Python 3.10+
- Node.js 18+（部分工具需要）
- 一个 API Key（从中转站获取）

## 中转站配置

本课程所有示例通过中转站调用 LLM API：

```
中转站地址：https://onekey.dualseason.com
```

中转站兼容 OpenAI SDK 格式，只需修改 `base_url`：

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-你的密钥",
    base_url="https://onekey.dualseason.com/v1"
)
```

在 LangChain 中使用：

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="deepseek-v4-flash",
    api_key="sk-你的密钥",
    base_url="https://onekey.dualseason.com/v1"
)
```

## 安装依赖

```bash
# 创建虚拟环境（推荐）
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

# 安装依赖
pip install langchain langchain-community langchain-openai langgraph crewai openai python-dotenv
```

## 环境变量

创建 `.env` 文件：

```bash
OPENAI_API_KEY=sk-你的密钥
OPENAI_API_BASE=https://onekey.dualseason.com/v1
OPENAI_MODEL_NAME=deepseek-v4-flash
```

## 验证安装

运行以下代码验证配置是否正确：

```python
from openai import OpenAI
import os
from dotenv import load_dotenv

load_dotenv()

client = OpenAI(
    api_key=os.getenv("OPENAI_API_KEY"),
    base_url=os.getenv("OPENAI_API_BASE")
)

resp = client.chat.completions.create(
    model=os.getenv("OPENAI_MODEL_NAME"),
    messages=[{"role": "user", "content": "你好"}]
)
print(resp.choices[0].message.content)
```

如果看到正常回复，说明配置成功！
