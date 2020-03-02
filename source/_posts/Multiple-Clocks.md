---
title: Multiple Clocks
date: 2020-03-02 16:10:50
tags:
 - STA
categories:
 - 书籍文档
 
---



多时钟交互路径如何检查建立时间和保持时间。

<!--more-->

## Integer Multiples

设计中经常会定义出现成整数倍数关系的时钟。这样的例子中，我们在相关的时钟（related clocks）中找出公共的时钟周期来进行STA分析。

> two clocks are related if they have a data path between their domains)

Here is an example that shows four related clocks:

```tcl
create_clock -name CLKM -period 20 -waveform {0 10} [get_ports CLKM]
create_clock -name CLKQ -period 10 -waveform {0 5}
create_clock -name CLKP -period 5 -waveform {0 2.5} [get_ports CLKP]
```

当分析CLKP和CLKM时钟域时，公共的时钟基本周期为20ns，如下图所示。

*Integer multiple clocks*

![STA5.1](/../sliu0827.github.io/source/_posts/Multiple-Clocks/STA5.1.png)

> Expanding clock 'CLKP' to base period of 20.00 (old period was
> 5.00, added 6 edges).
> Expanding clock 'CLKQ' to base period of 20.00 (old period was
> 10.00, added 2 edges).

简单分析可知，在 CLKP 15ns时刻出发至CLKM 20ns时刻捕获进行setup time check 最为严苛。而在CLKP&CLKM 0ns时刻进行hold time check最为合理。时序报告这里不再贴出。

## Non-Integer Multiples