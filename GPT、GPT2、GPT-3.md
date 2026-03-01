# 大模型架构

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
