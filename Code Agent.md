# Code Agent

+++

> 问题：
>
> verl是一个 agent RL训练框架，LocAgent

## 分工

**LocAgent**（`./LocAgent/`）是一个**代码定位 Agent 框架**，来自同名论文。它做三件事：

1. 把代码库解析成有向异构图（含 contains / imports / invokes / inherits 关系）并建 BM25 索引
2. 提供三个核心工具：`search_code_snippets`、`explore_tree_structure`、`get_entity_contents`
3. 提供独立的推理入口（`auto_search_main.py`）和 SFT 监督微调脚本（`sft_train.py`），可以脱离 RL 独立使用

**verl**（`/verl/`）是 ByteDance 开源的**分布式强化学习训练框架**，负责所有 RL 训练基础设施：PPO/GRPO 算法、Ray 分布式调度、Actor/Critic Worker 管理等。

+++

## verl

`AsyncLLMServerManager`负责管理与多个 OpenAI 兼容的 LLM 服务器之间的异步通信，并同时提供**负载均衡**和**粘性会话**两大核心功能。

- **负载均衡**：当有多个 LLM 服务器实例时，需要将请求均匀地分散到各个服务器上，避免单个服务器过载。
- **粘性会话**：在多轮对话中，后续的生成请求应该尽量发送到与第一轮相同的服务器上，以便利用服务器的**前缀缓存（prefix caching）**，从而加速推理。

为此，它内部维护了一个最小堆（用于实现最少请求数优先的负载均衡）和一个 LRU 缓存（用于记录每个 `request_id` 对应的服务器，实现粘性）。

