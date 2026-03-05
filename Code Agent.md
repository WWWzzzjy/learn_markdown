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

