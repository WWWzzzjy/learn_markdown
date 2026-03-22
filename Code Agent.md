# Code Agent

+++





> lora
>
> SGLang：Continuous Batching 
>
> vit Qwen（晚上）
>
> **FP8** 量化

## CodeAgent

+++

#### CodeAgent定位：

* issue localization：开发中，当开发者收到一个 Bug Report，第一步不是修复，而是找到需要修改的代码在哪里。对于大型代码库，一个 issue 描述的是自然语言，代码库是结构化程序，两者之间存在天然的语义鸿沟。
* Agent 的目标就是用一个基于图结构的 Agent 来解决多跳代码定位问题：把代码库建成有向异构图，让 Agent 沿着调用链、继承链在图上寻找真正需要修改的位置。
* 它的最终目标不是直接修 Bug，而是为下游的修复工具（如 Agentless、SWE-Agent）提供精准的定位输入，相当于在整个自动化软件工程流水线里专注做好"找到哪里"这一件事。

#### 使用图结构原因：

* 代码库里真正重要的关系，不是文件之间的层级关系、而是**函数调用、类继承、模块导入这些语义关系**，一个 Bug 的根源往往不在报错的地方，而在调用链的上游某个函数里。从报错位置到根源，需要跨越多个文件、沿着调用链追溯——这就是所谓多跳推理(multi-hopreasoning)问题。

* 传统的解析代码库的方法：

  1. 稠密的检索方法（Embedding 向量检索， 类似RAG）

     ​	把代码块用代码专用的 embedding 模型（如 CodeBERT、UniXcoder）向量化，存到向量数据库。用 issue 描述做 query，返回语义相似的代码块。

     ​	**局限**：语义相似不等于依赖相关。issue 描述的是用户可见的现象，真正需要修改的函数可能在五层调用之下，语义上和 issue 毫无关联。

  2. 生成式方法：利用长上下文，直接把整个仓库喂给LLM

     ​	**局限**：一是 token 消耗极大，成本高；二是 LLM 对超长上下文的注意力会衰减，中间的内容容易被忽视；三是真实 repo 动辄几十万行，根本放不下。

  3. Agent + 文件系统工具（如SWE-agent）：

     ​	给 LLM 配备 `open_file`、`search_file`、`grep` 等文件操作工具，让 Agent 像人一样手动导航仓库。这是 SWE-agent 等工作的做法。

     ​	**局限**：更重要的是 Agent 每一步都靠 LLM 的直觉决定"下一步打开哪个文件"，没有任何结构约束。利用图结构可以看到多跳内的所有依赖关系

     

#### 图工具介绍

把代码库解析成有向异构图（含 contains / imports / invokes / inherits 关系）并建 BM25 索引====

**node 类型：Directory、File、Class、Function**

**edge 类型：Contains、Invokes、Inherits、imports**

**实体EntityID命名规则：`"django/forms/forms.py:BaseForm.full_clean"`**

文件节点就是相对路径，冒号 `:` 分隔文件路径和代码实体，点号 `.` 分隔类名和方法名

1. `search_entity`：**搜寻节点**，用于"从关键词找到图里的节点"，返回完整实体ID。

   内部三成降级：

   * 精准的图节点匹配，返回代码块
   * 实体名称的倒排索引，返回完整实体ID
   * BM25在所有节点的"code"属性中做词频匹配，返回最相关的代码块

2. `retrieve_entity`：**精确读取工具**。在图节点中查询，返回节点代码块，若查找不到直接返回报错信息

3. `traverse_graph`：**依赖遍历工具**。从一个节点出发，在图上做 BFS，探索图上的结构关系，返回 **tree** 结构。可以设定多跳遍历的方向和深度。



#### 推理流程

1. **prompt 拼接输入：**

   * **system prompt（角色定位任务边界、工具说明）：**

     需要明确告诉模型它是一个**只读的代码分析专家，不修改文件，只定位**。

     同时要限定任务范围——**不是解决 bug，不是写补丁，只是找到需要修改的位置**。边界不清楚模型容易跑偏，比如开始分析修复方案而不是继续搜索。

     每个工具的功能描述、参数格式、返回值格式、适用场景。

   * **user prompt（带有CoT的高质量Few-shot实例）**

     需要显式告诉模型可以在单次回复里输出多个 tool_call，并给出示例。

   * **task instruction（任务说明，推理模板）+ Problem Statement（issue 具体内容）**

     给定完整执行 Localization 任务的框架：提取关键信息→定位模块→分析执行→找修改位置

     `<trace_locs>` 标签格式必须明确，包括每行一个实体、按重要性排序、使用完整路径等。格式不规范会导致 reward 计算时解析失败，轨迹白跑。

     **安全约束：**不该做的操作要显式禁止，比如不修改文件、不访问外部网络、不泄露系统信息等。这类约束如果只依赖模型的默认行为，在边界情况下容易被突破。

   

   

4. **建立推理入口**：

   1. 有明显关键词的issue（有报错位置）

      > File "django/forms/forms.py", line 185, in full_clean
      > AttributeError: 'NoneType' object has no attribute 'clean'

      直接从traceback中提取文件名和函数名作为第一步的搜索名

   2. 有复现bug代码

      > form = MyForm(data={'name': ''}) 
      >
      > result = form.is_valid()

      能从代码里提取函数名 、类名等。

   3. 纯文字表述

      > When using Django forms with custom validation, the error messages  are not properly propagated to the parent form when using formsets. The expected behavior is that all child errors should be visible  in the parent, but currently only the first error appears.

      从issue文本中提取涉及到的关键实体，先用traverse_graph看repo的目录结构，同时traverse_depth设置大一些，看看目录下的文件中哪些实体匹配名称

   ​	

#### 训练流程

加载 tokenizer 分词器；构建训练数据集（Rarquet格式）；加载 reward 函数；构建RayTrainer；初始化worker，fit() 启动训练

GRPO：

1. **Rollout**：generation，vLLM推理，输出轨迹（工具调用轨迹`toll_call_history`、response_mask、<trace_locs>)

   ​	Rollout 阶段，Agent 每轮工具调用结束后，除了把工具返回内容拼回对话

2. **Reward函数**：为了实行并行工具调用和解决 credit assignment，利用NDCG和tool efficiency作为奖励函数。为了控制 轮数限制，在reward中再加一个token长度惩罚项（token/token_budget）

   每次LLM调用工具返回的实体都记录下来形成一个tool 集合，然后每一次调用都计算当前返回的实体中有多少是在tool集合中没见过的，计算一个0~1的tool efficiency

   α·NDCG@5 + β（NDCG@5 · e）

   e = <img src="./assets/image-20260320152326507.png" alt="image-20260320152326507" style="zoom: 50%;" />

   $PReward = \Delta (NDCG)\cdot(1+\gamma\cdot e_t)-\lambda\cdot(token/token_{budget})$

   

   

   

   

   

### 入口文件 `auto_search_main.py`流程

```
main()
  └── localize(args)
        └── run_localize(rank, ...)         ← 多进程并发处理
              ├── set_current_issue(bug)     ← 初始化当前 issue 环境
              ├── 构建 messages（system prompt + task instruction）
              ├── auto_search_process(...)   ← 核心 Agent 循环（子进程）
              │     ├── LLM 调用（litellm）
              │     ├── 解析 response → Action
              │     ├── 执行工具 → 获得 Observation
              │     └── 循环直到 <finish>
              └── get_loc_results_from_raw_outputs()  ← 解析输出
  └── merge(args)
        └── merge_sample_locations()        ← 多次采样结果聚合（MRR/Majority）
```

* 使用`MRR(Mean Reciprocal Rank，平均倒数排序) / majority(多次投票)`对多次采样结果进行融合排序
* `Pass@k`表示对于每个问题，模型生成的 Top-k 的候选答案中，至少一个是与真实答案匹配的概率

### 工具

*  `SearchEntity`：根据关键字检索相关实体，如果关键字是Entity ID则直接图节点匹配；如果是名字，走BM25稀疏检索
* `TraverseGraph`：节点为起点，在图上做 BFS，输出树形的依赖关系文本。支持搜索方向（upstream/downstream/both）、深度、节点类型过滤、边类型过滤。
  * `batch_build_graph.py`：利用AST（抽象语法树）解析repo结构，返回为`networkx.MultiDiGraph`图数据结构并且保存为.pkl序列化成二进制文件
* `RetrieveEntity`：根据 Entity ID 拿出实体的完整代码

## [verl](https://zhuanlan.zhihu.com/p/1931076626940139506)

+++

**VeRL **是字节跳动 seed 团队和香港大学开发的专为大预言模型 post training 设计的高性能、可扩展的强化学习框架。它最核心的特点是采用了 **HybridFlow Controller（混合控制器流）** 的架构，融合**单控制器（Single-Controller）**和**多控制器（Multi-Controller）**，可更好实现和执行多种RL算法，显著提升训练吞吐量，降低开发和维护复杂度。

**设计目标：**在多进程计算环境中保持单进程式的编程简洁性。

**实现方式：**

- WorkerGroup 代理多进程 Worker，通过装饰器封装分布式逻辑。
- 控制器仅需调用高层 API，底层自动处理数据并行和结果聚合。

**Hybrid Flow**：RL的训练逻辑和 Pretrain/SFT 不一样，涉及到多个模型之间的交互和协作。VeRL 将 LLM RL 训练逻辑的 dataflow建模成一个两层的 hybrid flow 问题，进行了解耦，包括：

- **控制流**：位于high-level，使用**单进程**，描述了**多个模型角色之间的交互逻辑**，如actor make experience结束后，Critic、RM、reference开始计算分数，完成后计算 GAE 和相应 loss；

  中心进程同意调度的所有计算节点（worker）

- **计算流**：位于low-level，使用**多进程**，描述了单个**模型角色内部的计算流程**（如前向反向传播、优化器更新、自回归生成等），管理模型的训练和推理具体过程；

  下层每一个worker内部做高效的分布式计算

  <img src="./assets/v2-dacf92cf5850fe2e99d040ac5d8a41a0_r.jpg" alt="img" style="zoom:50%;" />
  
  > 控制器（Driver Process）在单个进程中运行，而执行器（Rollout, Actor, Critic）等Worker Process会放置在特定的资源组（Resource Pool）中在多个进程中运行。

### 1. 数据协议

`DataProto`是 `verl `中所有 `Worker `之间传递数据的统一容器，规定了所有模块间的数据流转方式。`DataProto`的设计目标是**为强化学习训练中复杂的异构数据流提供一个统一的表示和操作接口**，主要分为：

- **张量数据**：字段 `batch`，类型 `TensorDict`，如 `input_ids`, `attention_mask`, `logits` 等，需要在GPU上进行快速计算。
- **非张量数据**：字段 `non_tensor_batch`，类型 `dict[str, np.ndarry]`，如文本形式的 `prompts`、`responses`、`ground_truth`，或是工具调用的参数、结果等复杂结构。
- **元信息**：字段 `meta_info`，类型 `dict` ，如当前训练步数、模型配置、时间戳等用于跟踪和调试的上下文。

```json
{ // 一条DataProto数据
    // DataProto.batch 模型计算相关
    TensorDict(
        fields={
            "input_ids": Tensor(shape=torch.Size([64, max_seq_len]), device=cpu, dtype=torch.float32, is_shared=False),
            "attention_mask": Tensor(shape=torch.Size([64, max_seq_len]), device=cpu, dtype=torch.float32, is_shared=False),
            "position_ids": Tensor(shape=torch.Size([64, smax_seq_len]), device=cpu, dtype=torch.float32, is_shared=False),
            "raw_prompt_ids": Tensor(shape=torch.Size([64, max_prompt_len]), device=cpu, dtype=torch.float32, is_shared=False)
        },
        batch_size=torch.Size([64]),
        device=None,
        is_shared=False)

    // DataProto.non_tensor_batch 每条样本的附加字段
    {  
        "raw_prompt": [batch],
        "full_prompts": [batch],
        "index": [batch],
        "tools_kwargs": [batch]
    }

    // DataProto.meta_info 全局一些信号
    {
        "temperature": 0.7,
        "top_p": 0.8
    }
}
```



### 2. 分布式调度基础设施（`verl/single_controller`）

> 在传统的分布式训练（如使用PyTorch的DDP）中，你需要手动管理多个进程、处理进程间通信。

`single_controller`是基于 **Ray **构建，但将其底层的多 Actor 调用封装了起来，提供了一套更高级、更简洁的API。

```
single_controller/
├── base/
│   ├── worker.py          # Worker 基类，每个 GPU 进程上跑的对象
│   ├── worker_group.py    # WorkerGroup，对一组 Worker 的统一管理抽象，定义 ResourcePool
│   └── decorator.py       # @register_method 装饰器 + Dispatch/Execute 枚举
└── ray/
    └── base.py            # RayResourcePool, RayWorkerGroup, RayClassWithInitArgs
   
```

> **`decorator.py`** 是整个分布式调用机制的核心。它定义了两类枚举：
>
> - `Dispatch`：控制"如何把数据分发给 Worker"，如 `ONE_TO_ALL`（广播）、`DP_COMPUTE`（按 data-parallel 切分）、`DP_COMPUTE_DATA_PROTO`（切分 DataProto）
> - `Execute`：控制"Worker 返回后如何聚合"，如 `ALL_TO_ALL`、`RANK_ZERO`
>
> 被 `@register_method(dispatch_mode=..., execute_mode=...)` 装饰的方法，Driver 调用时会自动做数据切分、并发发送 RPC、收集结果聚合，完全透明。
>
> **`ray/base.py`** 把上述抽象用 Ray Actor 实现。`RayWorkerGroup` 管理一组 Ray Actor，`spawn()` 方法支持在同一批 GPU 上 colocate 多个角色（ActorRollout + RefPolicy 共享显存）。

* `ResourcePool`：**资源池**，管理跨多节点的计算资源（如GPU），记录每个节点上可用的**进程数量**和**GPU配置**

  * | **类名**             | **说明**                                                     |
    | -------------------- | ------------------------------------------------------------ |
    | `ResourcePool  `     | 基础资源池，管理 process_on_nodes 列表，提供 world_size、local_rank 等计算 |
    | ` RayResourcePool  ` | Ray  实现的资源池，支持 use_gpu=True，封装 Ray 的 PlacementGroup |

* `Worker`：Worker 是所有分布式计算节点的基类，每个 Ray Actor 都继承自 Worker。

  * WorkerHelper：提供获取节点 IP、空闲端口的静态方法。
  *  Worker： 基础 Worker 类，处理分布式初始化（rank、world_size 配置）、设备管理。
  * MegatronWorker：专为 Megatron 并行策略设计的 Worker，处理 TP/PP/DP 分组初始化。

* `WorkerGroup`：**它代表一组在分布式环境中的实际 Worker（封装一组同构的 Ray Actor），并提供统一的RPC 的调用接口去调用所有 Worker 的方法**。VERL实现了WorkerGroup绑定了一组 Worker 的方法，能通过在控制流中一次调用，触发关联的多个Worker执行远程任务

  

  ```python
  class WorkerGroup:
      def generate_sequences():
          chunks = dispatch_fn(batch, n=len(self.workers))
          futures = [
              worker.generate_sequences.remote(chunk)
              for worker, chunk in zip(self.workers, chunks)
      	]
          results = ray.get(futures)
  ```

  

  | **类名**                 | **说明**                                                     |
  | ------------------------ | :----------------------------------------------------------- |
  | `WorkerGroup`            | 基础 WorkerGroup，管理多个 Worker 实例，支持 Dispatch/Execute 装饰器批量调用 |
  | `RayWorkerGroup`         | 基于 Ray 的 WorkerGroup 实现，支持 spawn、init_model 等分布式初始化流程 |
  | `ClassWithInitArgs  `    | 延迟实例化的包装器，存储构造参数，在 Worker 节点上远程实例化 |
  | `RayClassWithInitArgs  ` | Ray  版本的  ClassWithInitArgs，用于 Ray Actor 远程构建      |

* `Dispatch / Execute` 装饰器：被装饰的方法，Driver 调用时会自动做数据切分、并发发送 RPC（Remote Procedure Call）、收集结果聚合，完全透明。

  * `@register(dispatch_mode, execute_mode)`：标记 Worker 方法，指定数据如何分发（scatter / broadcast）以及函数的执行方式。

  

```python
# Worker Process类：Worker 和 WorkerGroup
"""
每个 WorkerGroup 负责管理一组远程运行的 Worker。WorkerGroup 在其构造函数中启动进程。WorkerGroup 作为控制器进程的代理，用于与一组 Worker 进行交互，每个 Worker 都在 GPU 上运行，以执行特定的计算任务。为了实现这一功能，VERL实现了WorkerGroup绑定了一组 Worker 的方法，能通过在控制流中一次调用，触发关联的多个Worker执行远程任务
"""
```

**当一个 Worker 类的方法需要暴露为分布式调用时，开发人员会在方法定义上加上 `@register` 装饰器，并指定调度模式：**

当我们创建 `WorkerGroup` 时（例如通过 `ResourcePool` 实例化一组 Ray Actor），`WorkerGroup` 会遍历底层 Worker 类的所有方法，并执行以下操作：

- 对于每个被 `@register` 装饰的方法，检查其 `__register_info__`。

- **动态生成一个“代理方法”**，并将其绑定到 `WorkerGroup` 实例上。这个代理方法内部封装了完整的分布式调用逻辑。

- 这个动态生成的代理方法，其签名与原始 Worker 方法完全一致，但执行时会自动处理数据分发、并行执行和结果收集。

  > ![img](./assets/v2-ef5bfc192020b8a4fa9b1a7a86c570f9_r.jpg)
  >
  > WorkerGroup Worker 调度执行图

**单个 Process（WorkerGroup 管理下的 worker 组）内部数据处理：**

1. `dispatch_fn` (分发函数)：根据指定的模式，将输入数据拆分或复制，分发给组内的每一个Worker。

2. `execute_fn` (执行函数)：定义在各个Worker在管理的节点上的GPU上函数的执行方式（通常是并行执行）。

3. `collect_fn` (收集函数)：收集所有Worker的执行结果，并将其合并或整理成最终输出。？？

   <img src="./assets/v2-9a2b2ee3a961ce0ece7c9bcb44b73a63_r.jpg" alt="img"  />

**==核心调度模式==**：`single_controller` 支持多种数据分发模式，最常用的两种是 `ONE_TO_ALL` 和 `DP_COMPUTE_PROTO`

| 模式                   | `dispatch_fn` (分发)                                         | `collect_fn` (收集)                                          | 典型应用场景                                                 |
| :--------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **`ONE_TO_ALL`**       | **广播**：将相同的输入参数完整地复制并发送给每一个Worker。   | **简单拼接**：将所有Worker的输出简单合并成一个列表。         | 当所有Worker需要基于**相同的信息**执行任务时，例如每个GPU上都有一份完整的模型副本，并同时用不同的数据计算梯度。 |
| **`DP_COMPUTE_PROTO`** | **分片**：将一个大 `DataProto` 数据对象切分成多个小份，每个Worker获得其中一份。 | **合并**：将各个Worker处理后返回的小份 `DataProto` 重新合并成一个完整的 `DataProto`。 | 当数据量巨大，例如在生成（Rollout）阶段，将提示（prompts）数据分发给不同的GPU并行生成响应。 |

#### 底层基础建设：

- **`ResourcePool`** 负责维护全局资源视图，记录每个节点上有多少 GPU、可用端口等。它不直接参与方法调用，而是为 `WorkerGroup` 的创建提供资源分配依据。
- **`Worker` 的实例化**：`WorkerGroup` 在创建时，会利用 `ClassWithInitArgs` 在远程节点上真正启动 Ray Actor。每个 Actor 的初始化参数（如模型配置、rank 等）由 `ResourcePool` 分配的 index 决定。

> | 并行策略            | 切分维度 | 核心思想                                                    | 通信模式                                       | 典型问题                                         |
> | :------------------ | :------- | :---------------------------------------------------------- | :--------------------------------------------- | :----------------------------------------------- |
> | **数据并行 (DP)**   | 数据     | 每个GPU拥有完整模型副本，每个batch切分数据批次并行处理。    | 训练阶段间步（All-Reduce）同步梯度。           | 显存冗余（每个GPU都存一份完整模型）。            |
> | **张量并行 (TP)**   | 层内参数 | 将单层网络的计算任务（如矩阵乘法）切分到多个GPU上共同完成。 | 每层计算内同步（All-Reduce等），通信极其频繁。 | 通信开销极大，对卡间带宽要求高（通常需NVLink）。 |
> | **流水线并行 (PP)** | 层间     | 将模型的不同层分配到不同GPU，数据像流水线一样依次流过各层。 | 点对点（P2P）通信，仅相邻阶段传输数据。        | 存在GPU空闲等待的"流水线气泡"（Bubble）。        |



### 3. 训练引擎

```
trainer/
├── main_ppo.py            # 主入口，Hydra 解析配置，初始化所有组件，调 fit()
├── ppo/
│   ├── ray_trainer.py     # RayPPOTrainer，fit() 训练循环的实现
│   ├── core_algos.py      # 所有 RL 算法的数学实现（advantage, policy loss 等）
│   ├── reward.py          # compute_reward(), load_reward_manager()
│   ├── metric_utils.py    # 计算并收集训练 metrics（token 数、KL、clip ratio 等）
│   ├── utils.py           # Role 枚举，need_critic() 等辅助函数
│   └── rollout_corr_helper.py  # Rollout correction，IS weights 等 off-policy 修正
├── config/
│   ├── config.py          # 全局配置 dataclass（AlgoConfig、TrainerConfig 等）
│   └── algorithm.py       # 算法相关配置
├── fsdp_sft_trainer.py    # 单机 SFT 训练器
├── sft_trainer_ray.py     # 分布式 Ray SFT 训练器
├── main_generation.py     # 独立推理入口（不训练，只生成）
└── evaluate.py            # 评估逻辑


旧架构：
  fsdp_workers.py (Worker)
      ├── 手动初始化 FSDP module + optimizer
      ├── DataParallelPPOActor(module, optimizer)  ← 封装 PPO 算法逻辑
      └── DataParallelPPOCritic(module, optimizer) ← 封装 value 算法逻辑

新架构：
  engine_workers.py (Worker)
      └── engine (FSDPEngine / MegatronEngine)     ← 封装一切底层：模型+优化器+前向反向
              ↑ 通过 set_loss_fn() 注入具体的 PPO/value loss 函数
```

> `core_algos.py` 值得单独说：所有 RL 算法的数学核心都在这里，用注册机制管理：
>
> - Advantage 估计：GAE、GRPO、RLOO、REINFORCE++、ReMax、OPO、GPG 等 10 种
> - Policy Loss：vanilla（标准 PPO clip）、GSPO、SAPO、GPG、clip-cov、kl-cov、geo-mean、CISPO 等 8 种
> - 两个维度正交组合，不改 Trainer 代码就能切换算法
>
> **`engine` 把旧架构里手散在外面的那些事情全收进来了——模型加载、FSDP 包装、optimizer 构建、前向反向传播、权重同步**

训练引擎在 verl 中的核心设计定位是：**为上层算法提供一套无需关心具体并行策略的统一训练接口**。

它的目标是：让你在实现PPO、GRPO等算法时，无需关心模型底层是用了FSDP、Megatron还是其他后端，也无需操心数据是如何在GPU之间切分的

* 核心机制：**SPMD** 模式与 **WorkerGroup** 的协作：

  训练引擎以**SPMD（单程序多数据）**模式运行。这意味着，所有Worker（每个GPU通常对应一个Worker）执行的都是同一份代码，但处理的是不同的数据分片。

  - **在RL训练任务中**：训练引擎被封装在Worker内部，由`single_controller`模块中的`WorkerGroup`进行管理。当`RLTrainer`调用`worker_group.update_actor(data_proto)`时，`single_controller`会根据注册的调度模式（如`DP_COMPUTE_PROTO`）将数据分发给每个Worker，每个Worker内部的训练引擎则负责在本地数据上执行前向/反向计算，并参与梯度的全局同步。这一过程对上层算法完全透明。

* 核心抽象接口：**`BaseEngine`**所有引擎的抽象基类

* 多训练引擎后端实现：

  | 后端                 | 支持的并行策略                                              | 性能与适用场景                                               | 新模型支持时间    |
  | :------------------- | :---------------------------------------------------------- | :----------------------------------------------------------- | :---------------- |
  | **FSDP**             | FSDP (Fully Sharded Data Parallel) + SP (Sequence Parallel) | 对于稠密（Dense）模型性能不错，但对MoE模型效率较低。优点是支持HuggingFace模型**开箱即用**，适合快速迭代和中小规模训练。 | **第0天** (Day 0) |
  | **Megatron (MCore)** | DP+TP+PP+EP+CP (5D并行)                                     | **性能最佳**，专为超大模型（千亿甚至万亿参数）的极致优化设计。但集成新模型的成本也最高，通常需要数周。 | 数周或数月        |
  | **VeOmni**           | FSDP+SP+EP                                                  | 性能和模型支持度介于FSDP和Megatron之间，是一个平衡性选择，目前处于**Alpha测试阶段**。 | 约1周             |

verl同时支持和适配两套训练引擎，即FSDP和Megatron，前者逻辑清晰，且方便支持新的模型结构，research友好；而后者对超大规模（如100B以上）的模型训练更友好，并且参数resharding的开销更小，工程友好；

```python
# Driver Process 类：RayPPOTrainer
"""
RayPPOTrainer类扮演着Driver Process的角色， 它负责初始化并构建 Worker 和 WorkerGroup ，同时运行 PPO 算法的主循环，RayPPOTrainer 的 fit 函数是以单进程形式运行。WorkerGroup将在其指定的资源池（Resource Pool）上构建。资源池可以理解是 Ray 集群中的一组 GPU。
"""
```

#### 3D-HybridEngine 的协作

训练引擎是verl引以为傲的**3D-HybridEngine**（3D混合引擎）的核心组成部分。在Actor模型的训练中，3D-HybridEngine实现了**分时复用**：

1. **生成阶段 (Rollout Mode)**：训练引擎的状态（FSDP 分片权重、优化器状态、梯度）会被 **`Offload`（卸载）**到CPU，GPU 加载 vLLM 权重和KV cache；	

2. **训练阶段 (Training Mode)**：数据生成完毕后，释放生成引擎（KV Cache释放），训练引擎再从CPU**加载**回GPU，全力进行梯度计算和参数更新。

3. **代价是 PCIe 搬运时间**

   

### 4. VeRL 配置管理

VERL框架使用了Hydra来管理配置项，**Hydra** 是 Meta 开发的一个基于 Python 的开源配置管理工具，专为机

器学习等复杂项目设计。它通过**分层组合**的方式，解决了传统 `argparse` 或单一 `yaml` 难以管理成百上千参数的问题。

<img src="./assets/v2-bd93ca48aee896dd98b8803302ffe098_r.jpg" alt="img"  />

+++

`AsyncLLMServerManager`负责管理与多个 OpenAI 兼容的 LLM 服务器之间的异步通信，并同时提供**负载均衡**和**粘性会话**两大核心功能。

- **负载均衡**：当有多个 LLM 服务器实例时，需要将请求均匀地分散到各个服务器上，避免单个服务器过载。
- **粘性会话**：在多轮对话中，后续的生成请求应该尽量发送到与第一轮相同的服务器上，以便利用服务器的**前缀缓存（prefix caching）**，从而加速推理。

为此，它内部维护了一个最小堆（用于实现最少请求数优先的负载均衡）和一个 LRU 缓存（用于记录每个 `request_id` 对应的服务器，实现粘性）。

+++

**训练**：

传统数据并行（DDP）是每张 GPU 存一份完整的模型参数，不同 GPU 处理不同的数据批次，梯度同步时做 all-reduce（每个 GPU 都得到完整的汇总结果）。

FSDP 的解法是：**把参数、梯度、优化器状态都切碎，分散到所有 GPU 上**。

ZeRO-1 切优化器、-2 切优化器梯度、-3全切了，代价就是通信量比较大，每次传播都需要做all-gather（前向传播：拼出完整参数，计算） 和 reduce-scatter(归纳-分片，反向传播：先归纳得到完整梯度，然后再分片到各个GPU上)

vLLM用TP策略，把矩阵运算切开（attention head等）。最后all-reduce拼结果。原因：推理时每次只处理一个token 的KVcache，张量并行的方式推理速度快

MAP（Mean Average Precision，平均精度均值），跟NDCG相比就是分母会不会根据输出个数变化

MRR（Mean Reciprocal Rank，平均倒数排名）



**Meta RL:**

* 多语言的图构建
* 多语言的Benchmark 数据
* 
