# 大模型架构

+++

**GPT（2018）** 是奠基之作，核心思路是"无监督预训练 + 有监督微调"的两阶段范式。预训练阶段用大量无标注文本（BooksCorpus，约 800M 词）训练一个 12 层的 Transformer 解码器，学习通用语言表示；微调阶段针对具体下游任务（分类、蕴含、相似度、问答）加一个任务特定的输出头，配合少量标注数据训练。模型规模约 117M 参数。

**GPT-2（2019）** 的核心贡献是"语言模型本身就是多任务学习者"这一洞见。OpenAI 认为，足够大的语言模型在预训练时已经隐式地学习了各种任务，无需任何微调，仅凭自然语言提示（zero-shot）就能执行下游任务。训练数据扩大到 WebText（从 Reddit 高分链接爬取，~40GB），模型最大版本扩展到 1.5B 参数（48 层）。架构调整较小：Layer Norm 移到了每个子层输入端（Pre-LN），并增加了最终 self-attention 层后的 LN。

**GPT-3（2020）** 将规模推到极致：175B 参数，训练数据来自混合语料（Common Crawl、WebText2、Books1/2、Wikipedia），总计约 300B token。核心发现是 **in-context learning**：超大模型在不更新任何参数的情况下，仅凭 prompt 里的少量示例（few-shot）就能完成复杂任务，性能随参数量显著提升。报告同时对比了 zero-shot、one-shot、few-shot 三种设置，系统性地展示了涌现能力。架构沿用 GPT-2，增加了 alternating dense 与 locally banded sparse attention。

+++

## GPT

* 在无标号的数据集中训练一个预训练语言模型，在此基础上使用有监督微调

* 提出了自监督学习应用到NLP上

### 相关工作

* NLP领域中的自监督学习：
  * 词嵌入模型是怎么样的
* 无监督预训练
* 辅助训练目标

### 框架

两个阶段：

#### 无监督预训练(Unsupervised pre-training)

* 最大化对数似然函数：<img src="D:\ZJU\自学\GPT、GPT2、GPT-3.assets\image-20251118163324964.png" alt="image-20251118163324964" style="zoom: 80%;" />
* Transformer encoder：![image-20251118163435070](D:\ZJU\自学\GPT、GPT2、GPT-3.assets\image-20251118163435070.png)

#### 有监督微调（Supervised fine-tuning）

* 长度为m的序列和对应的标签y，将transformer最后一个block的输出经过一个线性层进行softmax操作：

  ![image-20251118192234199](D:\ZJU\自学\GPT、GPT2、GPT-3.assets\image-20251118192234199.png)

* 最大化目标函数：![image-20251118192258842](D:\ZJU\自学\GPT、GPT2、GPT-3.assets\image-20251118192258842.png)

* 将两个目标函数一起：![image-20251118192326597](D:\ZJU\自学\GPT、GPT2、GPT-3.assets\image-20251118192326597.png)

### 特定任务的输入转换

<img src="D:\ZJU\自学\GPT、GPT2、GPT-3.assets\image-20251119133147901.png" alt="image-20251119133147901" style="zoom:67%;" />

* 分类、蕴含、相似、多选题
* 开始符、间隔符、抽取符

+++

## GPT2

* 使用了zero-shot设定，而不去微调预训练模型
* 在GPT中，在下游任务中，对微调数据的输入进行了改造
* 开创了prompt engineering

+++

## GPT-3

### 训练数据

* 使用二分类对数据进行清洗
* 数据相似度分析（information retrieval）
* BERT、GPT、GPT2的数据融合在一起

### 模型训练

* 分布式训练

* 模型分割

* 数据分割

  > 带宽

### 模型评估

* 不同下游子任务有不同的评估方式
* 开放式：beam search

## 
