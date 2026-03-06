# DeepSeek-V3 技术报告

+++

## 技术特点

1. 借鉴了 Meta FAIR 团队论文：[Better & Faster Large Language Models via Multi-token Prediction](https://arxiv.org/pdf/2404.19737) 中的思路，使用了**MTP**（Multi-token Prediction）

   ![image-20260303135051639](D:\ZJU\自学\DeepSeek-V3.assets\image-20260303135051639.png)

​	$\mathbf{h}_i^{\prime k}=M_k[\mathrm{RMSNorm}(\mathbf{h}_i^{k-1});\mathrm{RMSNorm}(\mathrm{Emb}(t_{i+k}))],$

​	$\mathbf{h}_{1:T-K}^{k}=TRM_k(\mathbf{h}_{1:T-K}^{\prime k})$

​	$P_{i+k+1}^k=OutHead(h_i^k)$

2. 使用了 **DeepseekMoE**
   1. **MoE 存在的问题：**
   
      1. 知识冗余。基础知识在所有领域都要用到，被不同expert重复学习
      2. 知识杂糅。expert数量不够时，专业化程度不高。比如2个expert划分为文科理科，更多expert则可以划分为语文、数学、科学、地理等。
      3. 负载不均衡：只选择固定的几个expert完成任务，这样导致有些设备基本处于闲置状态，效率低下。
      4. 集群通讯问题。不同专家在不同设备上，计算速度远快于通讯。
         * 多机部署：MoE层的不同Expert线性层分布到不同设备，其他网络层则复制。
      5. batch size 减小。由于稀疏激活的原因，batch size会被切分，导致不是所有expert都接受batch size的数据。
   
   2. **DeepseekMoE 技术改进：**
   
      DeepSeekMoE在其他MoE工作的基础上，主要的两个改进：
   
      1. 细粒度专家划分。对Expert的粒度进行细分，提供更多样的expert激活组合，同时为了保持相同的计算消耗，减少每个expert的隐藏层维度
   
      2. 共享expert。对expert的类型进行划分，保留一部分expert对所有的输入都保持激活
   
      3. 负载均衡损失函数设计。
   
         1. 动态偏置调整
   
            - 实时监控每个专家被选中的频率
   
            - 如果某个专家过载，在计算分数时减去一个偏置值，降低其被选中的概率
   
            - 如果某个专家闲置，则加上一个偏置值，提高其被选中的概率
   
         2. 专家并行与设备限制
   
            - DeepSeek还将专家分布在多个GPU上，并限制单个Token激活的专家不超过3台设备。这样做的好处是：减少跨设备通信的开销、保证推理速度
