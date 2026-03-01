# MCP（Model Context Protocol 模型上下文协议）

+++

> 大模型本身只会问答，不会使用外部工具
>
> **MCP协议规定了如何发现和调用函数即 MCP Host 与 MCP Server 之间的交互，MCP就算脱离大模型也可以使用，MCP本身没有规定与模型的交互方式**

## MCP Host

Host更像是一个agent，会将用户的question和Server中已有的工具一起提供给模型

## MCP Server

启动程序：

* Python——uvx
* Dode——npx

Server给Host提供可用的Tool，大部分的Server都是通过输入输出来与Host进行沟通

## MCP Tool

Tool更像是函数，输入参数，返回结果



![image-20260224161731546](D:\ZJU\自学\MCP.assets\image-20260224161731546.png)