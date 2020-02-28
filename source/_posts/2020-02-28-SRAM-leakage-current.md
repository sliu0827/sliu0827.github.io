---
title: SRAM leakage current
categories:
  - 书籍文档 
tags:
  - Memory
date: 2020-02-28 15:31:49
---






*01.Leakage Currents in an SRAM of Conventional Design；*

*02.Gate-Tunnel Leakage and GIDL Currents*

<!--more-->



DRAM有较低的待机功耗，但是需要刷新操作，而SRAM则能静态的保存数据，不需要类似于刷新这样的特殊操作。因此，对于手机或移动设备中集成的SRAM芯片，我们很容易对其进行控制。此外，由于SRAM的制造工艺和CMOS工艺一样，例如移动蜂窝电话等中的应用处理器也使用SRAM作为集成的存储器。



随着MOS工艺的进步，近阈值泄漏电流（subthreshold leakage current）中会增加**gate-oxide tunnel leakage**和**gate-induced drain leakage（GIDL）currents**，成为主要的泄漏电流，增加的泄漏电流同时也增加了整个SRAM cell的静态漏电流。本小节将描述SRAM array的基本泄漏功耗。



# *leakage Currents in Conventional Design*

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) 

![memory3.1.png](https://i.loli.net/2020/02/28/zYtK3kj5GyNTQ6u.png)

Fig.1 Leakage currents in a technologically advanced nMOS device



关于工艺进步而造成泄漏电流增加的问题，图1展示了nMos晶体管在工艺上进步产生的4种主要泄漏电流。**包括gate-oxide tunnel leakage，GIDL，junction tunnel leakage以及subthreshold leakage currents.**



**Subthreshold leakage是当gate为off状态时，由drain到source的沟道泄漏电流，这种状态造成了在沟道区域的微弱反型层。**

![memory3.2.png](https://i.loli.net/2020/02/28/qmrVH6kYvJ3T175.png)



Fig.2 Standby leakagecurrents in an advanced technology SRAM cell of conventional design

 

图2中描述了随着工艺进步，使用传统设计方式的SRAM cell中的主要泄漏电流。在传统设计方式中，bit lines（BT，BB）和power line（VDDI）为1.5v，word line 和VSS 为0v。



在整个cell中，**2个gate-tunnel leakage currents，5个GIDL currents和3个subthreshold leakage currents** 组成了主要的漏电流**。**



**使用高阈值电压的MOS器件能够直接有效的减少subthreshold leakage currents ，虽然存储单元的读电流也会减小。**



**然而，通过简单的改变器件结构很难去减小tunnel leakage 和GIDL currents.**



# *Gate-Tunnel Leakage and GIDL Currents*



这小节描述了gate-tunnel leakage 和GIDL currents。在图3中给出了**gate-tunnel leakage current 随着栅压（gate voltage） 变化的曲线**，同时**栅氧化层厚度（Tox）作为一个参变量对比分析**。



**Tox 每增长2-°A，gate-tunnel leakagecurrent 会增长十倍**。随着工艺的进步，这种电流成为了泄漏电流的主要组成。



另一方面，**栅电流随着栅压减小而减小，减小电压0.5v（从1.5到1.0v）能减小这种泄漏电流的95%**。这是因为tunnel leakage current能随着栅氧化层的电场强度变化而简单的减小。



![memory3.3.png](https://i.loli.net/2020/02/28/KRAyngmhox23SPE.png)

Fig.3 Gate-tunnel leakage current as a function of gate voltage with the thickness ofthe gate oxide (Tox) varied as a parameter

 

图4描述了漏端电流由栅电压（VG）作用的电流曲线。**实线**是漏端电压（VD）为1.0V时的漏端电流曲线，**虚线**为1.5v时漏端电流曲线。当**VG大于0v**时，**subthreshold leakage currents 占主导地位**。当**VG小于0V**时**，GIDL currents 占主导地位**。



 

![memory3.4.png](https://i.loli.net/2020/02/28/7c3LNTH5Jml6xMI.png)

Fig.4 GIDL and subthreshold leakage currents



**在使用中提升阈值电压**，在VG=0v时subthreshold leakage currents 相比于GIDL current的强度是可以忽略的。





**GIDL current（IGIDL）对于电场强度F非常敏感**。GIDL current 对数曲线线性反比于栅压，栅压直接影响栅氧化层下的电场强度。GIDL current的大小由栅和漏的压差（VGD）决定，VGD由1.5减小到1.0v，电场强度减小，在器件中the GIDL current减小大约90%。



如果**阈值电压很低，**主要的泄漏电流是subthreshold leakage current。通过应用negative VGS，或者negativeVBB back-body bias voltage可以减小subthreshold leakage。具体实现电路和SRAM芯片架构下一期再接着介绍^_^





>  Ref:Low power and reliable SRAM memory cell and array design[M]. Springer Science & Business Media, 2011.