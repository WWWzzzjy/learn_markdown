# DPO（Direct preference optimization）

+++

## KL散度

<img src="D:\ZJU\自学\DPO.assets\image-20251210101204662.png" alt="image-20251210101204662" style="zoom:50%;" />

## BT模型（Bradley-Terry）  

<img src="D:\ZJU\自学\DPO.assets\image-20251210104753529.png" alt="image-20251210104753529" style="zoom:50%;" />

* Loss函数是用**极大似然估计**，取负对数似然作为Loss函数，加上指数函数就可以转换为二元交叉熵函数

<img src="D:\ZJU\自学\DPO.assets\image-20251210105017174.png" alt="image-20251210105017174" style="zoom:50%;" />



* 最后的DPO Loss：

  <img src="D:\ZJU\自学\DPO.assets\image-20251210105101269.png" alt="image-20251210105101269" style="zoom:50%;" />