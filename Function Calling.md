# Function Calling

+++

> 对于API、SDK、IDE的理解
>
> 长期记忆能力
>
> Qwen、Gemini、Deepseek 各自技术特点
>

## 定义

* 广义：本质上就是模型调用外部工具的一种能力，让模型根据用户的输入自主选择工具并且解析工具的执行结果。由大模型在对话中判断是否需要调用函数，并生成调用需要的参数，最终返回符合约定格式的函数调用请求
* 狭义：指大模型内部与API做了相关支持的一种能力，最早由OpenAI引入。
  * 通过SFT、RL，使大模型具备根据上下文正确选择合适函数、生成有效参数的能力
  * 模型API额外开放对Function Calling的支持

### 标准 Function Calling 请求格式

```python
# 调用时，除了 messages，还传入 tools 参数
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "帮我搜索 calculate_sum 函数"}
    ],
    tools=[  # ← 额外传入工具描述
        {
            "type": "function",
            "function": {
                "name": "search_code_snippets",
                "description": "搜索代码片段",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "search_terms": {"type": "array", "items": {"type": "string"}}
                    }
                }
            }
        }
    ]
)
```

### 标准 Function Calling 响应格式

```python
response.choices[0].message = {
    "role": "assistant",
    "content": None,          # ← 文本内容为空
    "tool_calls": [           # ← 结构化工具调用
        {
            "id": "call_abc123",
            "type": "function",
            "function": {
                "name": "search_code_snippets",   # ← 函数名
                "arguments": '{"search_terms": ["calculate_sum"]}'  # ← 参数（JSON字符串）
            }
        }
    ]
}
```

### Qwen 等早期开源模型，不支持FC协议，使用 XML 格式

```python
<tool_call>
{"name": "search_code_snippets", "arguments": {"search_terms": ["calculate_sum"]}}
</tool_call>

<execute_ipython>
search_code_snippets(search_terms=["calculate_sum"])
</execute_ipython>
```

