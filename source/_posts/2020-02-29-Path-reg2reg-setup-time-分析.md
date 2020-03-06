---
title: reg2reg setup time 分析
date: 2020-02-29 14:54:45
tags:
 - STA
categories:
 - 书籍文档
---



简单地对*寄存器到寄存器路径*建立时间的报告进行了分析

<!--more-->

## 理想的时钟网络

![STA2.1.png](https://i.loli.net/2020/02/29/ec9qmofu2OEhbXv.png) 

可以先分析出下面几个问题：

> 1.what are the start point and the end point of this path？
>
> 2.Which Path Group is this path belong to？
>
> 3.Path Type？
>
> 4.Constrain？

 分析可知，

- Start point: the launch flip-flop UFF0;End point: the capture flip-flop UFF1

- Path group:归属于CLKM

- Path Type ：max，意味着报告中所有路径区最大延时，对应setup time check

- Constraints:

  ```tcl
  create_clock -name CLKM -period 10 -waveform {0 5} [get_ports CLKM]
  set_clock_uncertainty -setup 0.3 [all_clocks]
  set_clock_transition -rise 0.2 [all_clocks]
set_clock_transition -fall 0.15 [all_clocks]
  ```
  
  > 设置 clock uncertainty可以加紧约束

总结报告，对于flip-flop path：

- launch path： 时钟由源点出发，经过Tlaunch = 0，经过flip-flop CLK2Q 延时Tck2q = 0.16，经过组合逻辑延时Tdp=0.04+0.05，最终到达时序路径终点UFF1/D，Data arrival time = 0.26；

- Capture path：时钟源点加上一个时钟周期，经过理想的时钟网络，最终到达终点UFF1/CK。data required time 还需要减掉 clock uncertainty 和 library setup time，得到data required time = 9.66；

- Slack = data required time - data arrival time，slack 大于0则满足约束，具体公式总结如下图所示。

![STA2.2.png](https://i.loli.net/2020/02/29/UavV19MIKOT32cR.png)

## 时钟网络有一个确定的延时值

上面报告中clock network delay是一个理想的值（marked as ideal），这意味着clock trees被认为是理想的，即clock path上所有的buffer delay都为0。显然，当时钟树被建立之后，clock paths会先显示真实的delays（marked as propagated），如下图所示。

![STA2.3.png](https://i.loli.net/2020/02/29/efwkadL7CAlOmNx.png) 

**后一个时钟为0.12比前一个触发器的时钟要晚到一点，差值012-0.11=0.01称之为时钟的*偏斜*。**

**另外时序报告中标有“r”和“f”，分别代表clock或数据信号的rising and falling edge。**因为是建立时间检查，所以通常取上升或下降沿延迟中的最大延迟值。 

### 两种类型的clock latencies

*clock source latency & clock network latency*

- Clock source latency：也称insertion delay，是由时钟源传播至当前分析时钟定义点所花费的时间。

  Source latency 不会影响内部设计的时序分析，因为它被同时加到launch clock 和capture clock。

  ```tcl
  set_clock_latency -source -rise 0.7 [get_clocks CLKM]
  
  set_clock_latency -source -fall 0.65 [get_clocks CLKM] 
  ```

- Clock network latency：如果不使用 -source 命令，命令定义的则为clock network latency，是由DUA中时钟定义点到flip-flop clock pin的时间。

![STA2.4.png](https://i.loli.net/2020/02/29/xhVU1BYlvqLASFe.png)
