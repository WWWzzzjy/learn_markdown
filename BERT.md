# BERT：Pre-training of Deep Bidirectional Transformer for Language Understanding

+++

BERT之前的工作

* ELMo
* GPT

task：

* 文本分类（Text Classification）
* 命名实体识别（Named Entity Recognition, NER）
* 问答系统（Question Answering）
* 语言推理（Natural Language Inference, NLI）

## ==预训练模型做特征表示的两类策略==

* 基于特征：ELMo，特征与输入一起输入
* 基于微调：GPT

二者在做训练的时候都用一个目标函数：单向的语言模型（预测）

## MLM

> 对于句子和对词元的任务有啥区别
>
> 目前大预言模型有哪些tasks

## 计算参数量

## 输入和输出表征

使用WordPiece进行嵌入

* 区分句子对：

  * 使用[SEP]
  * 在输入中（token embedding加上position embedding通过学习得来的）加上一个句子的嵌入（是在句子A还是B）

  

## 预训练

### Task1：Masked LM

* 训练数据中15%的`token`使用掩码`[MASK]`代替进行预测
* 对这15%概率被选中的词，有80%真实这样操作；10%概率替换成随机词元；10%概率不操作

### Task 2：NSP（next sentence prediction）

> 为了应对一些下游任务，例如QA

* 训练数据中有50%概率选择两个句子没有联系（负例），50%两个句子是原文相邻（正例）

### 训练数据



## 微调

