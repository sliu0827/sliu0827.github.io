---
title: Balance in circuit design
date: 2020-02-28 15:35:57
tags:
 - Memory
 - SRAM
categories:
 - 书籍文档

---

*01.EFR Scheme for Low Power SRAM；*

*02.Chip Architecture；*

*03.Results；*

*04.Source Line Voltage Control Technique*

<!--more-->

本文主要介绍一种新的设计架构来获得更低的静态功耗，电源电压控制技术（source line voltage control techniques）。传统设计中，power line的电压为某一常数值来保证SRAM稳定工作。在此，新方法中通过控制power line电压来改善漏电状况。



首先，描述了在手机低电压SRAM cell中使用的**electric field relaxation（EFR）**电路。**采用更高的阈值电压，有助于减小SRAM cell的静态功耗**。



其次，描述了在移动蜂窝电话应用处理器中集成的SRAM**减小漏电流的设计技术**。**采用较低的阈值电压，有助于获得SRAM cell更低的获取时间（low access time）**。



# *EFR Scheme for Low Power SRAM*



EFR电路如图1所示，它能从电路级别消除泄漏电流。**在保持阶段（retention mode），对于bit lines（BT and BB）用1.0V的供电替换原先1.5V的供电，同时VSSM从0提升到0.5V**。

![memory4.1.png](https://i.loli.net/2020/02/28/mRG8OLKn5vijHZy.png)

Fig. 1 Electric field relaxation scheme



**黑色的实线箭头表示bit lines（BT and BB）从1.5V替换为1.0V的差异**。这种relaxation能减小gate leakage和GIDL currents大约90%。



**灰色的实线箭头表示负的0.5v VGS应用到transfer nMOS晶体管**。负偏压能够将subthreshold leakagecurrent降到接近0。



**圆点的箭头表示负的0.5V back-bodybias电压VBB应用到driver nMOS晶体管**。这个反偏压能减小subthreshold leakage current大约90%。



以上是MOS管的source电压和衬底之间电压由0变为0.5V的区别，但是，the band-to-band tunneling leakage仍然没有改善。



图2给出了在25℃（Fig.2a）和90℃（Fig.2b）温度环境下，对传统设计方式和EFR设计方式的SRAM cells在最坏的工艺环境下（the worst-case process）静态泄漏电流的测试数据。每种器件类型合计了subthreshold和GIDL currents。

![memory4.2.png](https://i.loli.net/2020/02/28/Qg7JOSoHqxdKilb.png)Fig. 2 Measured SRAM cell leakagecurrent; (a) 25ıC/worst case (b) 90ıC/worst case



在两个例子中，pMOS的阈值电压为1.0V，在25℃和90℃中两者的subthreshold leakage均比GIDL currents小得多。nMOS的阈值电压为0.7V，subthreshold leakage current相对于GIDLcurrent在25℃更小，但在90℃更大。cell的静态泄漏功耗在EFR设计中25℃时为16.6fA，90℃时为101.7fA，分别相比于传统设计的cell小82.5%和91.8%。



# *Chip Architecture*



![memory4.3.png](https://i.loli.net/2020/02/28/j5faidzXmlgFVbc.png)Fig.3 Circuit diagram of one mat



图3展示了SRAM芯片某一簇（mat）的电路图，整个SRAM chip 有32 mats。每个mat由2048 word lines构成的256-data-bit和20-parity-bit columns组成，且每个mat被划分为4个bank。



读操作的仿真时序图在Fig4中展示。该SRAM是异步的，如果地址信号不发生改变SRAM将保持retention mode，**如果地址信号改变，SRAM由retention mode转换为active mode，产生一个ATD 脉冲，激活reset 操作**。





**在4.3ns的reset期间，选中的bank中memory cell的VSS line（VSSM）从0.5V下拉到0V，同时bit-line pairs（BT,and BB）从1.0V预充为1.5V**。



传统异步SRAM中通常是在reset time时，bit-line pair由0v预充为1.5V，在我们的EFR电路设计中，我们可以容易的在reset time期间为bit-line预充和VSSM放电。另外，由于EFR电路和asynchronous SRAM上述类似，access time不会增加。

![memory4.4.png](https://i.loli.net/2020/02/28/8HLKbmp5lkYrAcW.png)

Fig. 4 Simulated waveforms for readoperation



**随后word line（wl）激活。随着sense amplifiers的激活，选中的128 bits of data 和10 parity bits出现在local bus（LBUS）**。为了减少对power supplies噪声的产生，VSSM被控制在bank level而不是在mat level。这种控制方法可以保持VSS的噪声低于32mV，这个结果在仿真中通过power supply nets可以观察得到。VSSM line由第三层金属加强来以控制VSSM从0.5到0V。在完成读操作的500us内，VSSM line非常慢的恢复到0.5V。在读操作完成的100ms内由漏电流作用，bit-line pairs容易的恢复回1.0V。因此，复位操作很难影响到动态功耗。



# *Results*



![memory4.5.png](https://i.loli.net/2020/02/28/NiBFqxsgVILSGDM.jpg)

Fig. 5 Photograph and features of thechip



# *Source Line Voltage Control Technique*



本节描述聚焦于移动电话应用处理器的减小漏电流技术。通常，集成在应用处理器中的SRAM会使用较低的阈值电压。因此，主要的泄漏电流是subthreshold leakage current。为了减小SRAM cells在未选通状态下的泄漏电流，便提出了source line voltage control 技术，来控制powerline电压，并保持最小限度地牺牲操作稳定性。



**SRAM的Power supply voltage控制为VDD和VSS的中间电位，中间电位电压同时减小泄露电流以及数据保持电流**。中间电压的产生由一个简单的电路实现，因为如果采用复杂电路，会有额外的功耗和面积代价。**这一节介绍一个简单的电压控制电路，包括电压切换选择，电阻和二极管，此电路可以降低额外的功耗和面积代价**。

![memory4.6.png](https://i.loli.net/2020/02/28/ALVXdw1abKkBSMJ.png)Fig. 6 Leakage and stability vs. localpower line voltage



SRAM模块必须满足其性能目标，设计能实现足够稳定的保存数据。控制VSSM的电压来减小memory cell的leakage时，更高的VSSM能减小SRAM cell leakage到一个更小的级别。另一方面，也能破坏SRAM的数据保持稳定性。这样的关系在图6中描述。为了满足low leakage 和high stability，VSSM必须被控制到能满足leakage目标的最低电压。图7展示了VSSM控制电路，**PLVC1，由3个NMOSFETS组成。一个NMOSFET作为在VSSM和VSS之间的电压切换（MS1），一个作为二极管（MD1），另一个作为电阻（MR1）**。MR1有一个长的栅极且保持常开。

![memory4.7.png](https://i.loli.net/2020/02/28/CxKvrTI1EJUaH5d.png)

Fig. 7 VSSM controller in embedded SRAM



由130nm制造工艺生产的memory，其cell的主要leakage current 是subthreshold current，很大程度受阈值电压变化的影响。例如，Vth减小100mV，subthreshold leakage增加超过十倍，而MOSFET的驱动电流仅增加10%。



**当制造的MOSFET 阈值电压Vth波动到一个高值，memory cell的leakage会大大减小。**

**因为MR1为开启状态，电流通过MR1轻微的减小。如果MR1的电流比memory cell leakage电流大，VSSM电压变低。**



**当阈值电压Vth波动到较低的值，VSSM为高，但是MD1能限制VSSM上升，保持为某一电压。这样保持足够低的电压来保持存储的数据。**



图8展示了MOSFET制造的阈值电压Vth和VSSM电压的对比。横轴展示了阈值电压Vth设计的理想值和生产制造的差异，受工艺变化而变化。当制造晶体管的阈值电压Vth低，leakage current高，VSSM 电压必须为高来减小leakage current。当制造晶体管为高，leakage current 为低，VSSM电压需要保持低来确保SRAM的retention 稳定性。

![memory4.8.png](https://i.loli.net/2020/02/28/qeHMDoxYJZLtTGN.png)

Fig.8 VSSM voltage when Vth is changed



虚线描绘了满足leakage 目标且最大化cell 稳定性的理想情况。如果MOSFET为worst-leakage 器件，MOSFET的阈值电压Vth低于设计值0.1V，VSSM需要高出0.3V来满足leakage目标。



**如果voltage controller仅有power switch和diode组成，VSSM电压如图中（a）线所示**，能满足leakage 目标，但是memory cell的稳定性降低。



**如果voltage controller PLVC1由constant-voltage source 电路构成，VSSM电压如图中（b）线所示**，能满足leakage 目标，且稳定性要好于线（a），但是memory cell的稳定性仍然低。



因此，**使用三种类型的MOSFETs，MS1，MD1和MR1，作为voltage controller，VSSM电压（图8中粗线）趋近于理想值**。这些值能满足leakage目标，且保持了更高的稳定性。PLVC1能同时满足leakage目标且保持SRAM高的稳定性。



![memory4.9.png](https://i.loli.net/2020/02/28/GkUdsolePiMtHc3.png)Fig.9 Effect of leakage reduction in1-Mbit leakage-worst embedded SRAM



在图9中，陈述了1-Mbit SRAM（图8） 的leakage current。使用leakage降低技术，leakage current能减少大约90%。在图10中给出了130nm SRAM 芯片模型的特征和照片信息。

![memory4.10.png](https://i.loli.net/2020/02/28/d4pgR9kxEJUic1j.jpg)Fig. 10 Photograph and features of theprototype chip





> *Ref**：**Low Power and Reliable SRAM memory Cell and Array Design*