# DeepSeek-V3 技术报告

+++

## 技术特点

1. 借鉴了 Meta FAIR 团队论文：[Better & Faster Large Language Models via Multi-token Prediction](https://arxiv.org/pdf/2404.19737) 中的思路，使用了**MTP**（Multi-token Prediction）

   ![image-20260303135051639](D:\ZJU\自学\DeepSeek-V3.assets\image-20260303135051639.png)

​	$\mathbf{h}_i^{\prime k}=M_k[\mathrm{RMSNorm}(\mathbf{h}_i^{k-1});\mathrm{RMSNorm}(\mathrm{Emb}(t_{i+k}))],$

​	$\mathbf{h}_{1:T-K}^{k}=TRM_k(\mathbf{h}_{1:T-K}^{\prime k})$

​	$P_{i+k+1}^k=OutHead(h_i^k)$

