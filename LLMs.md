# LLMs

+++

## 基本介绍

### LLMs 基本模型

* BERT
* GPT
* LLaMa
* FLAN-T5
* BLOOM
* PaLM

## Transformers架构

![image-20251020103628027](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20251020103628027.png)

![image-20251017105759744](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20251017105759744.png)

> 编码器和解码器；解码器模型为什么输入和输出必须等长？为什么GPT等模型是仅解码器模型？
>
> Encoder是BERT模型和RAG应用的基础

### 架构演变

![image-20251020104041621](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20251020104041621.png)

* Bag-of-Words模型：只是把词元向量化了，没有考虑文本本身的意义
  ![image-20251020104707517](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20251020104707517.png)

* Word2Vec：生成静态嵌入，不考虑上下文联系

  ![image-20251020105619354](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20251020105619354.png)

### 注意力编码和解码上下文（==注意力机制如何作用？==）

![image-20251020135347220](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20251020135347220.png)

![image-20251020135412433](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20251020135412433.png)

* 自回归
* 往往输出效果更好

### Transformer

* 不适用 RNNs 架构；加速训练，相较于 RNNs 能够实现并行化训练
* encoder 和 decoder 由多个引入注意力机制的块堆叠而成，增强编码器和解码器的强度

**编码器**：旨在表示文本并且在生成嵌入向量方面表现出色

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20251020140850216.png" alt="image-20251020140850216" style="zoom:50%;" />

* 输入生成随机嵌入向量，不使用word2vec；self-attention处理嵌入并更新（能够包含更多的上下文信息）

* 再通过前馈神经网络，生成上下文强联系的 token or word embeddings

* 自注意力是一种机制，并不是处理两个单独的序列，而是处理单个序列即与自身的比较

  <img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20251020140608936.png" alt="image-20251020140608936" style="zoom:50%;" />

**解码器**：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20251020141130991.png" alt="image-20251020141130991" style="zoom:50%;" />

* 取已经生成的词，传递给掩码自注意力（类似编码器）

* 生成的中间嵌入向量与编码器生成的嵌入向量一起传递给另一个注意力网络，同时处理生成的信息和已有的信息。最后通过神经网络生成序列中的下一个词

* 掩码自注意力：类似于自注意力，去除了上三角矩阵中的值（==本质意义是为了掩盖未来的位置，已有词元只能关注之前出现的词元==）

  <img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20251020145509167.png" alt="image-20251020145509167" style="zoom:50%;" />

  表示模型（Representation models）和 生成模型（Generative models）

  BERT（Bidirectional encoder representations from Transformer）：双向编码器表示的Transformer新架构，拓展不仅限于翻译工作。

  <img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20251020145959806.png" alt="image-20251020145959806" style="zoom:50%;" />

  输入包含了一个额外的词元 CLS，分类词元，用作输入嵌入，用于对特定任务进行微调

  

### 分词（BPE，Byte Pair Encoding）

BPE 是一种简单的数据压缩算法。BPE也是典型的基于subword的tokenization算法



* 输出 token probably，此时需要选择**解码策略**
* greedy search：总是选择概率最高的的作为下一个输出 token，即；贪婪解码，temperature=0
* beam search：在解空间中进行**宽度优先搜索 (BFS)**，但引入了**剪枝 (Pruning)** 策略，设定束宽来保持计算可行性
* Top_P解码：temperature>0，这就是为什么相同输入的情况下输出会不一样，这与解码策略有关
* Transformer使用并行计算，当生成一个token时并入提示进行循环计算
* KV caching：使用缓存来加速计算



> 词嵌入和位置编码
>
> 词嵌入形成512维度向量，每一个维度都可以理解为一个特征，例如与人的相关度、是否是水果，有些特征是很抽象的，不是能用特定语言来描述
>
> 位置编码
>
> 多头相较于单头，多头可以更加专注于一类特征，而单头一下子包含了所有特征
>
> 残差连接
>
> Query Key Value 三类矩阵的具体含义是什么
>
> 具体怎么解决并行训练问题还是不太懂啊，好像有点懂了，是因为以往训练都是依靠上文结果作为输入，现在是直接进行上下文处理吗

## 提示工程 Prompting and prompt engineering 

* Prompt

* Inference

* Completion

* **ICL(In-context learning)**
  
  大语言模型在预训练阶段从大量文本中学习潜在的概念。当运用上下文学习进行推理时，其借助任务说明或演示示例来**“锚定"**其在预训练期间所习得的相关概念从而进行上下文学习，并对问题进行预测。
  
  * zero-shot inference(零样例推断)
  * one-shot inference(单样例推断)
  * few-shot inference(少样例推断) 

零样例推断在大模型中实现很好，但是对于一些小模型，在单样例或者少样例推断下还是无法表现的很好的情况下， 就需要进行模型微调

样例的选择的两个主要依据：**相似性**和**多样性**

## 配置参数

Max new token

top_k：选择softmax后概率最大的k个token进行随机取样

top_p（top cumulative probability）：选择概率最大之和小于top_p的token进行随机采样，top_p越大，随机选择的token就越多，意味着创造性就越强。top_p的存在是因为由于vocabulary非常大，有几十万个词，因此通过softmax计算出来的会有非常多的低概率的与前文毫无关系的垃圾词，称之为**长尾词**

Temperature：越大，随机性越大；越小，随机性越小。是一个缩放因子，在模型的最后一个Softmax层中应用，影响下一个Token的概率分布形状。不同于top_k/p，调整temperature会改变模型的预测。因为变得是概率，top类参数不改变概率大小，只是一种选择方式改变

![image-20251029235811498](D:\ZJU\自学\LLMs.assets\image-20251029235811498.png)

## 生成式AI的项目生命周期

<img src="D:\ZJU\自学\LLMs.assets\image-20251030090742348.png" alt="image-20251030090742348" style="zoom:50%;" />

##  指令微调（指令提示）

> 全面微调和参数微调
>
> PEFT
>
> LoRA

标准的交叉熵函数来计算两个token分布质检的损失

<img src="D:\ZJU\自学\LLMs.assets\image-20251030094526610.png" alt="image-20251030094526610" style="zoom:50%;" />

## 对单一任务进行微调

* Catastrophic forgetting 灾难性遗忘：全面微调下对全局参数进行的调整，虽然会对特定任务的表现提高，但是对其他任务的性能会降低

如何避免灾难性遗忘：

* 多任务微调
* 这就提出了PEFT微调，它保留了原始LLM权重（预训练），只训练少量特定任务的适配器层和参数

## 多任务指令微调

* Instruction fine-tuning with FLAN

## 评估模型

> 首先，有一些专门的术语：gram，一个词是unigram，两个词是bigram，n个词是n-gram
>
> 主要的指标跟普通机器学习差不多：召回率Recall，精确度Precision，F1

* ROUGE：用于评估文本摘要

  ROUGE-1是对unigram进行评估，很容易出错例如生成的摘要中有`not`这些词

  ROUGE-2是对bigram进行评估

  ROUGE-L是对`LCS（Longest common subsequence）最长公共子序列`进行评估

  不同的任务下不能使用一个指标来评定，这样没有参考意义

* BLUE：用于评估文本翻译

  是对多个n-gram大小的平均精确度进行计算

## Benchmarks

使用已有的数据集建立模型评估基准

* Evaluation benchmarks：GLUE、SuperGLUE、HELM、MMLU、BIG-bench

  > 学习一下各个基本的评价指标计算方式和意义

* GLUE：general language understanding evaluation
* MMLU：massive multitask language understanding（2021）
* BIG-bench：（2022）
* HELM：holistic evaluation of language models，在基本的准确性度量之外进行评估，包含公平性fairness、偏见bais、有害信息toxicity等

## 参数高效微调方法：PEFT

特点：

* 大部分甚至所有的LLM权重都被冻结，训练参数较少只有原始的15-20%
* 不容易出现灾难性遗忘的问题
* 完全微调会产生一个新版本的模型，适用于训练的每一个任务，每一个产生的模型大小都与原始模型一样大，所有会产生存储问题，而PEFT改善了这个问题
* PEFT trade-offs：存储效率memory efficiency、参数效率parameter efficiency、训练速度training speed、模型表现model performance、推理成本inference costs
* 三类PEFT methods：
  1. Selective：微调LLM参数的子集，只训练某些组件或者特定层甚至单个参数类型
  2. Reparameterization（LoRA）：使用原始LLM参数，通过创建原始网络权重的新的低秩转换来减少训练参数
  3. Additive（Adapters，Soft Prompts）：保留所有LLM训练参数，并引入新的可训练组件

### LoRA（Low-Rank Adaptation）低秩适应

* 冻结所有原始模型参数

* 注入一对低秩分解矩阵

* 训练这对矩阵的权重

  

* 在推理过程中，让两矩阵相乘创建一个与冻结矩阵相同维度的矩阵

* 加到原始矩阵中



* 不同秩对于LoRA的影响不同，当Rank增加后，对于模型效果的提升或者说是损失的提升达到饱和，不在提升，所有选择一个合适的秩是一个模型性能提升和计算量的平衡过程

### 软提示 Soft Prompt

 <img src="D:\ZJU\自学\LLMs.assets\image-20251112104225253.png" alt="image-20251112104225253" style="zoom:50%;" />

## RLHF（Reinforcement learning from human feedback）

<img src="D:\ZJU\自学\LLMs.assets\image-20251112150212013.png" alt="image-20251112150212013" style="zoom:50%;" />

* RL algorithm：
  * PPO（Proximal Policy Optimization）
  * Q-learning

### 奖励攻击

* 使用一个冻结权重的原始LLM模型作为参考模型与RL更新后的LLM模型进行比较，使用KL散度计算衡量两个分布的不同，然后添加一个惩罚项到奖励计算中

