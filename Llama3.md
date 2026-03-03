# Llama3

+++

> 待学习内容：
>
> CoT
>
> Function Calling到底是一个什么概念

## 架构创新

### 提出了激活函数：SwiGLU 代替 ReLU



### 提出 RMS Norm 代替 Layer Norm

* RMSNorm 的计算更简单，减少了均值计算的开销，加速了推理和训练。RMSNorm是在Layer Norm之上的改进，它通过舍弃中心不变性（不计算均值）来降低计算量。

  $\bar{a}_i=\frac{a_i}{\mathrm{RMS}(\mathbf{a})}g_i,\quad\mathrm{where}\ \mathrm{RMS}(\mathbf{a})=\sqrt{\frac{1}{n}\sum_{i=1}^na_i^2}.$

  

###  旋转位置编码 RoPE（Rotary Position Embedding）



### GQA（Group Query Attention）



> Instruction fine tuning 是不是跟 SFT一个概念

### 训练过程

<img src="D:\ZJU\自学\Llama3.assets\image-20260303105032597.png" alt="image-20260303105032597" style="zoom:50%;" />

