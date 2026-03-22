# 累积增益（Cumulative Gain, CG）

+++



# 折损累计增益（[Discounted cumulative gain](https://zhida.zhihu.com/search?content_id=118397824&content_type=Article&match_order=1&q=Discounted+cumulative+gain&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NzM0NjQ1ODAsInEiOiJEaXNjb3VudGVkIGN1bXVsYXRpdmUgZ2FpbiIsInpoaWRhX3NvdXJjZSI6ImVudGl0eSIsImNvbnRlbnRfaWQiOjExODM5NzgyNCwiY29udGVudF90eXBlIjoiQXJ0aWNsZSIsIm1hdGNoX29yZGVyIjoxLCJ6ZF90b2tlbiI6bnVsbH0.LQmWmfYxxE8oiwmWezBMIntulcFAufI7nwMo-k9HeRM&zhida_source=entity)，DCG）

+++

对于列表长度不一的情况无法进行比较，因此引入NDCG



# 归一化折损累计增益（[Normalized Discounted Cumulative Gain](https://zhida.zhihu.com/search?content_id=118397824&content_type=Article&match_order=1&q=Normalized+Discounted+Cumulative+Gain&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NzM0NjQ1ODAsInEiOiJOb3JtYWxpemVkIERpc2NvdW50ZWQgQ3VtdWxhdGl2ZSBHYWluIiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MTE4Mzk3ODI0LCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.oLoDfuqczt0w_2dtOsrPjqwGXKqj8EGCi9c27Vi-WFs&zhida_source=entity), NDCG）

+++



# MAP（Mean Average Precision，平均精度均值）

+++

![image-20260321110758266](./assets/image-20260321110758266.png)

**含义**：不仅关心有没有命中，还关心命中的位置是否靠前。

**例子**：真实答案是2个文件，预测顺序是[✅, ❌, ✅, ❌, ❌]

- 第1位命中：P(1) = 1/1 = 1.0
- 第3位命中：P(3) = 2/3 ≈ 0.67
- AP = (1.0 + 0.67) / 2 = 0.835

# MRR（Mean Reciprocal Rank，平均倒数排名）

+++

![image-20260321110820585](./assets/image-20260321110820585.png)

其中 $\text{rank}_i $ 是第 i 个 issue 中 **第一个**正确答案的排名位置。

**含义**：只关心**第一个**正确答案出现在第几位，位置越靠前分越高。

**例子**：

| issue | 第一个正确答案位置 | Reciprocal Rank |
| ----- | ------------------ | --------------- |
| 1     | 第1位              | 1/1 = 1.0       |
| 2     | 第3位              | 1/3 ≈ 0.33      |
| 3     | 未命中             | 0               |

MRR = (1.0 + 0.33 + 0) / 3 = 0.44

# Acc（Accuracy）— 准确率

+++

![image-20260321110538085](./assets/image-20260321110538085.png)

**含义**：在预测的前 K 个结果中，是否包含了至少一个正确答案。

**例子**：预测了5个文件，真实答案在第3位 → Acc@5 = 1，Acc@2 = 0



# Recall（召回率）

+++

![image-20260321110624892](./assets/image-20260321110624892.png)

**含义**：在预测的前 K 个结果中，覆盖了多少比例的正确答案。

**例子**：真实答案是3个文件，前K个预测里命中了2个 → Recall@K = 2/3
