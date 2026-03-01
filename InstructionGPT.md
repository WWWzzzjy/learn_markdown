# InstructionGPT(2022.3.4)

+++

<img src="D:\ZJU\自学\InstructionGPT.assets\image-20251124211334751.png" alt="image-20251124211334751" style="zoom:50%;" />

## Introduction

* 语言模型的**目标函数**是没有align的，因为网上那些文本数据语言模型预测下一个词，但是这个跟人类期望生成的内容是有差异的
* 期望语言模型目标：helpful、honest、harmless

<img src="D:\ZJU\自学\InstructionGPT.assets\image-20251124100609524.png" alt="image-20251124100609524" style="zoom:50%;" />

### 步骤：

* step1：利用prompt和人工回答对GPT-3模型进行有监督微调SFT（人工成本高）
* step2：利用step1预训练好的模型对prompt生成四个回答，人工标注四个回答好坏排序，训练一个RM模型
* step3：继续去微调SFT，使得生成的回答能够得到高分数

+++

## Relative works





+++

## Dateset

* prompt：

   ![image-20251124103617224](D:\ZJU\自学\InstructionGPT.assets\image-20251124103617224.png)

  利用第一批的prompt预训练模型让用户使用再收集prompt（根据用户进行数据集的切割，因为同一个用户的提出的prompt）

+++

## Model

### SFT

* 在GPT-3模型上进行微调，使用了**余弦学习率衰减策略**
* 由于数据集少，训练一个epoch就过拟合，但是多训练周期有利于后续强化学习

### RM（6B的GPT-3模型）

* 把前面的GPT-3模型的softmax去掉，用一个线性层输出一个一维标量

* 利用Logistic Regression（Sigmod输出概率结合二元交叉熵函数，这里没使用二元交叉熵）计算损失：Pairwise

  ![image-20251124184435171](D:\ZJU\自学\InstructionGPT.assets\image-20251124184435171.png)

### RL

![image-20251124210423914](D:\ZJU\自学\InstructionGPT.assets\image-20251124210423914.png)

* KL散度：
* 假如了GPT-3训练的目标函数（防止灾难性遗忘）

