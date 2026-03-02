# Transformer

+++

![img](https://pic2.zhimg.com/v2-f6380627207ff4d1e72addfafeaff0bb_1440w.jpg)



这里有一些新的感悟：就是在计算当前时刻下token的向量值时，是根据当前时刻前所有token的V加权求和再加上残差连接计算出来的，即就是在初始输入下进行了语义上偏离，这是根据前文的语义计算出来的当前文本情景下的token的向量。

* 解码器中，是一个一个输出的，这种形式的输出叫做自回归（auto-regressive），意思就是过去时刻的输出也是这个时刻的输入

* batch norm 和 layer norm 的区别：

  <img src="D:\ZJU\自学\Transformer.assets\image-20251103150546754.png" alt="image-20251103150546754" style="zoom:50%;" />

  由于每一个样本的大小不一样，小批量batch的均值和方差抖动较大：<img src="D:\ZJU\自学\Transformer.assets\image-20251103150731367.png" alt="image-20251103150731367" style="zoom:50%;" />

  layer norm 是计算每个样本的均值和方差，也不需要计算全局的均值和方差

* 解码器带掩码的注意力机制是为了训练的时候看不到t时间之后的输入，保证训练和预测的行为是一致的

* attention只注意各个token之间的相关性，比如你输入的顺序打乱，其实输出的是一样的QKV，所以要加入时序信息也就是positional encoding

* 训练中的正则化：Residual Dropout、Label Smoothing


### embedding

* 我们需要把文字转换为token再转换为计算机能看懂的向量，如果选择传统的目标识别那种one-hot的稀疏向量，确实可以实现，但是这样的向量都是正交的，无法建立向量之间的关系。
* 因此，选择使用稠密向量，每个维度的无法解释，但是我们可以形象的去理解。
* 通过在向量空间中的“距离”，可以定义两个向量之间的关系

### Positional Encoding

![image-20251110162417965](D:\ZJU\自学\Transformer.assets\image-20251110162417965.png)

* 角频率w按几何级数递减，既能表示局部，又能覆盖长距离

* 特点：

  ![image-20251110162616934](D:\ZJU\自学\Transformer.assets\image-20251110162616934.png)

### 对于Mask并不是很理解

* padding mask
* causal mask

### attention

* 本质上就是观察其他token在句子中对自己的影响大小，同时来调整自己的语义

+++

## encoder-decoder

交叉注意力机制
