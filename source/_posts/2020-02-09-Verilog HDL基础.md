---
layout: post
title: Verilog HDL 基础
date: 2020-02-09
categories:
	- 程序设计
tags: 

	- RTL
---



# Verilog HDL 语言的特点

**互连（connectivity）**：网络数据类型表示结构实体（例如门）之间的物理连接，不能存储值，由驱动器（例如门或连续赋值语句，assign）驱动。常见类型包括wire型和tri型。

**并发**（concurrency）：可以有效地描述并行的硬件系统 

**时间**（time）：定义了绝对和相对的时间度量，可综合操作符具有物理延迟

**可综合**：always、if-else、case、assgin

**不可综合**：function、for、fork-join、while

<!--more-->

# 常见可综合语法与硬件电路的关系
## if-else相关语句的硬件结构映射及优化

对应的硬件结构是多路选择器（Multiplexing Hardware）

- 需要根据输入约束，小心设计：先“加”后“选”，先“选”后“加”

![RTL1.1.png](https://i.loli.net/2020/02/27/DCQPkFWzhmJA9SZ.png)

- 单if语句，无优先级的判断结构

- 多if语句，具有优先级的判断结构

    最后一级选择信号具有最高优先权；
    具有优先级的多选择结构会消耗组合逻辑

![RTL1.2.png](https://i.loli.net/2020/02/27/9tMhjpIADVRfw5W.png)

##  case相关语句的硬件结构映射及优化

对应的硬件结构是无优先级的判断结构，与单if语句的区别在于各条件互斥，多用于指令译码电路

##  慎用latch

- 综合器很难解释latch，因此，除非特殊用途，一般避免引入latch。

  ![RTL1.3.png](https://i.loli.net/2020/02/27/6GzFjdIWsUPpKye.png)
  latch由电平触发，非同步控制。DFF由时钟边沿触发，同步控制。

- 在使能信号有效时，latch相当于通路，无效时latch保持输出数据的当前值。因此latch容易产生毛刺（glitch），DFF则不易产生毛刺。latch将静态时序分析变得极为复杂。
   一般的设计规则是：在绝大多数设计中避免产生latch。latch最大的危害在于不能过滤毛刺，这对于下一级电路是极其危险的，所以只要能用DFF的地方，就不用latch。

- 易引入latch的途径：使用不完备的条件判断语句

- 防止产生非目的性latch的措施： 
  	使用完备的if…else语句
    	为每个输入条件设计输出操作，为case语句设置default操作
    	仔细检查综合器生成的报告，latch会以warning的形式报告

- 综合器指令，full-case和parallel-case

  Full-case：告诉综合器，当前case结构所列条件已完备

  Parallel-case：告诉DC，所有条件互斥，且并行，无优先权

## 逻辑复制，均衡负载

通过逻辑复制，降低关键信号的扇出，进而降低该信号的传播延迟，提高电路性能。

![RTL1.4.png](https://i.loli.net/2020/02/27/pB7qAltnWrg6vK3.png)

## 资源共享，减小面积

![RTL1.5.png](https://i.loli.net/2020/02/27/FzgvMmDeUKl6BiT.png)

## 资源顺序重排，降低传播延时

![RTL1.6.png](https://i.loli.net/2020/02/27/lXg6WIJ89CjOSPH.png)

## 同步复位与异步复位

## 少用“：？”赋值语句

使用always用于逻辑运算，关于assign，仅用于信号连接，难以阅读，且多层嵌套后很难被综合其解释

# 可综合风格
## 完整的always敏感信号列表

所有的组合逻辑或锁存的always结构必须有敏感信号列表。这个敏感信号列表必须包含所有的输入信号

*原因：综合过程将产生一个取决于除开敏感信号列表中所有其他值的结构，将可能在行为仿真和门级仿真间产生潜在的适配。*

## 每个always敏感信号列表对应一个时钟

在综合过程中，每个Verilog always敏感信号列表只能对应一个时钟。

*原因：这是将每一个过程限制在单一寄存器类型的要求。*

## 在时序电路中必须使用非阻塞赋值（<=），组合逻辑电路必须使用阻塞赋值（=）。

# 模块划分
## 分开异步逻辑与同步逻辑

建议分开异步逻辑与同步逻辑

*原因：避免综合时的问题，简化约束和编码难度。*

## 分开控制逻辑和存储器

建议控制逻辑和存储器逻辑分成独立的模块

*原因：便于高层的存储器模块的使用和便于重新描述为不同的存储器类型。*

> 参考：[芯动力--硬件加速设计方法](https://www.icourse163.org/course/SWJTU-1207492806?tid=1207824209 "芯动力--硬件加速设计方法")