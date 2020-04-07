---
title: FIFO 介绍2
date: 2020-03-26 21:39:10
tags:
 - 数字IC
categories:
 - 数字IC设计基础
---



如何对FIFO最小深度计算和FIFO空满标志判断的漏洞分析。

<!--more-->



## FIFO 深度计算

-----

### 例题分析

例题：假设FIFO的写时钟为100MHZ，读时钟为80MHZ。在FIFO输入侧，每100个写时钟，写入80个数据；读数据侧，假定每个时钟读走一个数据。

*请问FIFO深度设置为多少可以保证FIFO不会上溢出和下溢出？*



**分析清楚数据轻载和重载时数据的传输任务**

![FIFO4.1.png](https://i.loli.net/2020/03/26/4XNaLU5bHIoDgn3.png)

如图所示，列出了100个写时钟周期中数据，写入的不同情况。

当第一次100个写时钟内，数据的写入发生在后80个时钟中，且第二次100个时钟内，数据的写入发生在前80个时钟。这种情况下，会有连续的160个时钟周期在写入数据，负荷最重，即是需要分析的重载情况。

**分析：**

假设写入时为最坏情况（背靠背），即在160*（1/100）微秒内写入160个数据。

写入所需要的时间是:

```
Time:burst_cycle * t_wclk = burst_cycle/f_wclk = 160/100
```

该时间段内能读出的数据：

```
data_num_read = Time/t_read = Time/(1/f_read) = Time * f_read = (160/100) * 80
```

FIFO应该设置的深度值：

```
depth = burst - data_num_read = 160 - (160/100) * 80 = 32
```

### 问题一般化

参数为：

- 写时钟频率：WCLK；
- 读时钟频率：RCLK；
- 写数据时，每B个时钟周期内会有A个数据写入FIFO；
- 读数据时，每Y个时钟周期内会有X个数据读出FIFO。

FIFO的最小深度应为：

```
depth = burst_length - burst_length/wclk * (rclk * (X/Y))
```

> burst_length / WCLK 表示burst持续时间；
>
> RCLK *（X/Y）表示读的实际速度，两者乘积为读数据量；
>
> burst_length 表示写入数据量。



## FIFO空满标志的判断方法是否存在漏洞？

----

判断方法回顾：

- rptr同步到wclk时钟域后，在wclk时钟域与wprt进行比较，生成FULL标志；
- wptr同步到rclk时钟域后，在rclk时钟域与rprt进行比较，生成EMPTY标志。

假设读写时钟的频率接近，如下图所示。

![FIFO4.2.png](https://i.loli.net/2020/03/26/1y2pAFSzTu4HfkQ.png)

判断满状态时，假设将读指针rptr 数据“3”通过电平同步器同步至写时钟域。经过两个wclk时钟周期后，数据到达写时钟域，经wptr比较后相等生成FULL信号。**即在该时刻，FIFO认为已满，且满时的读指针为“3”。实际上，该时刻读指针已经为“5”，FIFO并不是真的满状态。**

**结论：**

- 对于FULL信号的生成机制，同步后的读地址一定是小于或者等于当前的读地址，所以此时判断FIFO为满不一定为真满，更保守；

- 对于EMPTY信号的生成机制，同样成立，判断为空时不一定为真的空状态；

- 异步FIFO通过比较读写地址来进行空满判断，读写地址的同步机制使得FIFO的空满状态仍然留有余量，存在一定的冗余空间。

------

*Reference*： 

1.[FIFO 介绍1]( https://www.sliu.info/2020/FIFO-介绍1/)

2.[芯动力—硬件加速设计方法](https://www.icourse163.org/course/SWJTU-1207492806?tid=1207824209)

3.[Simulation and Synthesis Techniques for Asynchronous FIFO Design](http://www.sunburst-design.com/papers/CummingsSNUG2002SJ_FIFO1.pdf)

