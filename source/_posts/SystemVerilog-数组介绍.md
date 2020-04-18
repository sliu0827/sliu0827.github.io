---
title: SystemVerilog 数组介绍
date: 2020-04-11 14:11:35
tags:
 - SystemVerilog
categories:
 - 芯片验证基础

---

什么是组合型数组？什么又是非组合型数组？

<!--more-->

## 组合型和非组合型数组

-----

### 非组合型（unpacked）

verilog中通常定义例如`reg [15:0] RAM[0:4095];`，这种数组定义为memory array用作数据存储。SV将这种声明数组的方式称作非组合型数组，即数组中的成员之间存储数据都是互相独立的。

例如作以下定义：

```verilog
wire [7:0] table [3:0];
```

对应的存储结构为：

![SV数组介绍3.1.png](https://i.loli.net/2020/04/11/PvsCz36pwLSZYrm.png)

### 组合型（packed）

SV中将具有向量结构的数组声明方式作为组合型。其相对于非组合型在声明时的差别在于，其索引在数组名的左侧，数组维度从左到右来识别。

例如作如下定义：

```verilog
logic [3:0][7:0] data;//2维组合声明
```

对应的存储结构为：

![SV数组介绍3.2.png](https://i.loli.net/2020/04/11/kFmCXt6IansPUY1.png)

### 区别分析

初始化时，组合型（packed）存储值是连续的，可以对所有元素统一赋值；非组合型（unpacked）中则需要分别对每一个维度进行赋值。

例如：

```verilog
//packed
logic [3:0][7:0] a = 32’h0; // vector assignment
logic [3:0][7:0] b = {16’hz,16’h0}; // concatenate operator
logic [3:0][7:0] c = {16{2’b01}}; // replicate operator

//unpacked
int d [0:1][0:3] = ’{ ’{7,3,0,5}, ’{2,0,1,6} };
// d[0][0] = 7  d[0][1] = 3  d[0][2] = 0  d[0][3] = 5
// d[1][0] = 2  d[1][1] = 0  d[1][2] = 1  d[1][3] = 6

```

非组合型数组中，若两个数组进行拷贝和赋值时，左右两边数组的维度和大小必须严格一致；组合型数组中，赋值左右两侧操作数的大小和维度不同时，也可以做赋值，会采用截取或者扩展的方式进行赋值。

非组合型和组合型之间无法直接地互相赋值。

### 其他特性

- 添加了foreach循环结构，用于遍历一维或多维数组，不需要指定该数组的大小。

- 添加了系统函数，例如：

  `$dimensions(array_name)`,返回数组的维度；

  `$left(array_name,dimensions)`,返回指定维度的最左索引值；

  `$size(array_name,dimensions)`，返回指定维度的尺寸大小等等

## 其他数组类型

----

### 动态数组

动态数组在声明时使用“[]”，且不会为限制数组尺寸。动态数组一开始为空，使用new[]为其分配空间。同时，内建方法size(),delete()可以对动态数组进行操作。

### 队列

SV引入了队列，结合了数组和链表。通过“[$]”声明队列，索引值从0到$，常用的内建函数包括push_back(val)、push_front(val)、pop_back()和pop_front()来顺序添加或者移除并且获得数据成员。

### 关联数组

用于存放散列的数据成员，散列的索引类型除了为整形外还可以为字符串或者其它类型，散列的存储数据也可以为任意类型。

![SV数组介绍3.3.png](https://i.loli.net/2020/04/11/J8IzFPNGb3Z52W1.png)

## 数组操作方法

----

常见的包括以下几类：

- 缩减方法，sum、product、and、or和xor；
- 定位方法，min、max和unique等，foreach实现数组搜索或者find..width查找满足条件的数据成员；
- 排序方法，reverse、sort、rsort和shuffle等。

----

*Reference*： 

1.路科验证V0系列课程，可参考《芯片验证漫游指南：从理论到UVM的验证全视界》