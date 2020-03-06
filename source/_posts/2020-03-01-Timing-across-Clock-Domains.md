---
title: Timing across Clock Domains
date: 2020-03-01 12:52:38
tags:
 - STA
categories:
 - 书籍文档
---





快慢时钟交互路径如何检查建立时间和保持时间

 <!--more-->

# Slow to Fast Clock Domains

按下列命令定义Clock:

```tcl
create_clock -name CLKM -period 20 -waveform {0 10} [get_ports CLKM]
create_clock -name CLKP -period 5 -waveform {0 2.5} [get_ports CLKP]
```

*Path from a slow clock to a faster clock*

![STA4.1.png](https://i.loli.net/2020/03/01/x7XgV3cKHMfas9Y.png) 

当 launch flip-flop 和capture flip-flop时钟频率不同时，STA首先需要确定公共的周期

 *Setup and hold checks with slow to fast path*

![STA4.2.png](https://i.loli.net/2020/03/01/JFPeac2AMBQkV5p.png)

如上图所示，在快时钟的第一个时钟周期进行setup/hold检查，这也是最严苛的检查，有如下结果。

Setup time check 时序报告：

![STA4.3.png](https://i.loli.net/2020/03/01/xeDVXNYmaWhs7KP.png)


hold time check 时序报告：


![STA4.4.png](https://i.loli.net/2020/03/01/TkYIrztawg2l3M5.png)

但是实际情况中，组合逻辑延时可能会很大，远大于一个快时钟周期。因此用快时钟的第一个时钟周期来检查时序，是不合理的。根据实际延时，我们可以通过特殊指令，指定在后面第几个时钟周期进行检查。

```tcl
set_multicycle_path 4 -setup -from [get_clocks CLKM] -to [get_clocks CLKP] -end
```

> The -end specifies that the multicycle of 4 refers to the end point or the capture clock.

*Multicycle of 4 between clock domains*
![STA4.5.png](https://i.loli.net/2020/03/01/CxLa2IE5ijuNB8R.png) 


通过上述命令后，setup time check 在20ns时刻，hold time check 在15ns时刻。

Setup time check 时序报告：

![STA4.6.png](https://i.loli.net/2020/03/01/lzbyeaXTuxOPhQm.png)

hold time check 时序报告：

![STA4.7.png](https://i.loli.net/2020/03/01/6LAiWREQpydqMob.png)

在hold time check 的报告中，可以发现slack为负值，是违例的。在大多数设计中，数据的保持时间都比较短，所以在第一个时钟边沿，即launch edge 对应的时间边沿处检查是更能符合实际的。为实现这一点，我们需要hold time check做如下约束。

 ```tcl
set_multicycle_path 3 -hold  -from [get_clocks CLKM] -to [get_clocks CLKP] -end
 ```

 *Hold time relaxed with multicycle hold specification*

![STA4.8.png](https://i.loli.net/2020/03/01/CPNjQ5eBEvWzRUL.png)

hold time check 时序报告：

![STA4.9.png](https://i.loli.net/2020/03/01/ZDqOS7jEbTrPa5t.png)


总结一下，慢时钟到快时钟周期进行检查，可以设置在第N个快时钟周期来检查setup，同样也可以设置在第N个时钟周期对应的hold time check时间点的前N-1个时钟周期进行hold检查，这样更能贴合保持时间检查的实际情况。

# Fast to slow Clock Domains

 

通过下列命令，定义快时钟CLKP和慢时钟CLKM：

```tcl
create_clock -name CLKM -period 20 -waveform {0 10} [get_ports CLKM]
create_clock -name CLKP -period 5 -waveform {0 2.5} [get_ports CLKP]
```

Path from fast clock to slow clock domain如下图所示，存在4种setup timing checks possible，以最严格的setup4为例检查setup time。

![STA4.10.png](https://i.loli.net/2020/03/01/ofS3F1UvID4Ri9Y.png) 

Setup time check 时序报告：

![STA4.11.png](https://i.loli.net/2020/03/01/5OLJNHDsfRwPBEV.png)

最严格的保持时间检查是在快时钟的第一个时钟周期setup1 0ns时，确保capture edge 在0ns时不会捕获到launch flip-flop将要改变的数据。


![STA4.12.png](https://i.loli.net/2020/03/01/rwCXEFpYBSyj8fW.png)

同slow to fast clock domains setup/hold time check一样，我们可以通过设置约束调整快时钟的默认周期的数据发射边沿，设置其在第几个时钟周期进行建立时间检查，让慢时钟进行建立时间检查。 

```tcl
set_multicycle_path 2 -setup -from [get_clocks CLKP] -to [get_clocks CLKM] -start
set_multicycle_path 1 -hold -from [get_clocks CLKP] -to [get_clocks CLKM] -start
```

> The -start option refers to the launch clock and is the default for a multicycle hold.

 

As expected, the launch clock edge is at 10ns and the capture clock edge is at 20ns，The hold check is at 0ns where both the capture and launch clocks have rising edges.

*Setup multicycle of 2*

![STA4.13.png](https://i.loli.net/2020/03/01/Eh4GYrVkie2unDF.png)

