# 基础知识

+++

## Python库：multiprocessing

* `mp.process(target=worker, args, kwargs)`：创建一个子进程
  * `start()`：开跑
  * `join()`：主进程等待子进程结束
  * `worker(*args, **kwargs)`：子进程执行的函数
* `mp.queue()`：进程间通讯队列（IPC）
  * `.put()`：放入队列
  * `.get()`：默认阻塞，取不到就一直等
  * `.get_nowait()`：非阻塞式取任务
* `mp.lock()`：写文件锁，防止多个进程同时写文件/更改共享资源
* `mp.manager()`：启动一个“管理进程”
* `mp.spawn()`：批量启动 N 个进程（PyTorch 分布式/DDP 常用）

+++

## 类装饰器@

* `@classmethod`：将方法转换为类方法。类方法的第一个参数是类本身（通常命名为 `cls`），而不是实例（`self`）
* `@staticmethod`：将方法转换为静态方法。静态方法不会接受 `self` 或 `cls` 作为第一个参数，它就是普通的函数，通常不访问类或实例的属性。
* `@abstractmethod`：标记一个方法为抽象方法，表示该方法必须在子类中实现。它通常与 `ABC`（抽象基类）一起使用。
* `@property`：将方法转换为只读属性，使得可以像访问属性一样访问该方法，不用加`()`，但背后依然执行一些代码逻辑。
* `@dataclass`：
* `@register`：Python的注册器Registry提供了字符串到函数或类的[映射](https://zhida.zhihu.com/search?content_id=236726449&content_type=Article&match_order=1&q=映射&zhida_source=entity)，这个映射会被整合到一个字典中，开发者只要输入输入相应的字符串（为函数或类起的名字）和参数，就能获得一个函数或初始化好的类。

+++

## 库：uuid

UUID 库是用于在计算机系统中生成通用唯一标识符（Universally Unique Identifier）的工具库
