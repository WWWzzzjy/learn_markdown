# vLLM

+++

## KV cache 存在的显存占用问题：

在大模型推理时，按照可生成最长序列长度分配显存，利用率只有20%~40%，造成三种类型的浪费：

1. 预分配，但不会用到。（预分配了最长序列下的KVcache，但是实际上可能提前输出了终止符）
2. 预分配，但尚未用到。（显存会根据最长序列长度提前分配好KVcache，但是在生成前面token时后面的cache都没用到，但显存已经分配占用）
3. 显存之间的间隔碎片，不足以预分配给下一个文本生成。（虽然最大生成长度一致，但是prompt不同，每次预分配的KVcache也不同，因此在一个请求生成完毕释放缓存后，下一个较小的prompt的KVcache无法放入被释放的缓存中）



LLM推理速度有两个指标，分别是时延和吞吐。 

* 所谓时延，就是单条请求从发出到完成计算的时间，这个vLLM确实没有明显提高。
*  但是对于吞吐，是说服务器单位时间内完成了多少条请求的计算，因为优化了显存，可以增加单位时间内处理的请求数，所以吞吐会大幅增加



## Page Attention

### 优势：

* 按需分配，不提前分配
* 用Block来分配，减少了碎片大小
* 虚拟内存，方便实现调用

### 原理：

* 借鉴了操作系统中的虚拟内存和页管理技术：page作为分配内存的最小单元（4Kb），使用的进程分配不同的page

  <img src="D:\ZJU\自学\vllm.assets\image-20260303151335448.png" alt="image-20260303151335448" style="zoom:50%;" />

* page attention 将显存划分为不同的KV Block来管理KV Cache，llm每个请求需要的KV Cache被划分到不同的KV Block（每个Block中可以缓存的token的KV Cache数量是固定的， 一个token序列分配到的KV Block在物理显存中可以是不连续的）

  <img src="D:\ZJU\自学\vllm.assets\image-20260303152113006.png" alt="image-20260303152113006" style="zoom:50%;" />

* 对于后续生成的token的KV，会被继续加载到未被填满的Block中

* 对于每一个request，都有一个逻辑KV Cache（显存是连续的），然后vLLM维护一个**映射表**到物理显存的KV Cache

  <img src="D:\ZJU\自学\vllm.assets\image-20260303153554580.png" alt="image-20260303153554580" style="zoom:50%;" />

* Sharing KV Blocks：对于同一个prompt，希望生成n个Output。

  在物理显存KV Cache中，会标注每一个Block被逻辑内存中Block引用的次数，当引用数=0时，Block被占用的显存被释放。

  Copy on Write机制：当开始生成时，发现继续写入的Block的引用数为n时，触发机制。拷贝写入的Block到一个新的Block，然后各自开始生成，但是prompt起始部分的KV Block是共享的。

* 优化Beam Search中的显存占用，因为有共用token



