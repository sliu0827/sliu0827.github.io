---
title: SystemVerilog 类和包介绍
date: 2020-04-12 18:53:02
tags:
 - SystemVerilog
categories:
 - 芯片验证基础
---



类是一种可以包含数据和方法的类型。通过类的使用，实现面向对象编程（OOP，Object-Oriented Programming）......

<!--more-->

## 类的封装

----

类是一种可以包含数据和方法的类型。借助类的使用，实现面向对象编程（OOP，Object-Oriented Programming），用户可以创建复杂的数据类型，在更加抽象的层次建立测试平台和系统级模型，通过调用函数可以执行一个动作而不仅是完成电平的改变。

**OOP术语：**

- 类（class）：包含成员变量和成员方法；
- 对象（object）：类在例化后的实例；
- 句柄（handle）：指向对象的指针；
- 原型（prototype）：程序的声明部分，包含程序名、返回类型和参数列表

**如何创建一个类（class）？**

将class的声明和创建分开。

```verilog
Transaction tr; //声明一个句柄
tr = new(); //为一个Transaction对象分配空间
```

首先，为对象创建一个句柄，在声明时会被初始化为特殊值null。

接下来，调用构造函数new()函数来创建Transaction对象。new函数为Transaction分配空间，将变量初始化为默认值（二值变量为0，四值变量为X），并返回一个指向类对象的句柄，其类型就是类本身。

**赋值和拷贝**

- 浅拷贝（shallow copy）与深拷贝（deep copy）

  浅拷贝（shallow copy），在创建P2对象时，调用new函数进行复制，Packet 对象中的成员变量例如整数、字符串和句柄等被拷贝，但如果类中包含一个指向另一个类的句柄则不会被复制。

  ```verilog
  Packet P1;
  Packet P2;
  P1 = new;
  P2 = new P1;
  ```

  深拷贝（deep copy），对对象中的句柄指向的对象也做相应的拷贝，即做递归的拷贝。用户需要自己定义深拷贝的函数才能实现这种需求。

  例如有下列代码，

  ```verilog
  class rgb;
      byte red;
      byte green;
      byte blue;
  endclass:rgb
  
  class pixel;
      int x;
      int y;
      
      rgb color;
  endclass:pixel
  pixel dot = new;
  ```

  ![SV类和包介绍4.1.png](https://i.loli.net/2020/04/13/bkE2rxFoIqsuCej.png)

  图中(a)，利用new函数拷贝对象时，只能对color句柄做拷贝，而不会对其指向的对象也做拷贝，因此拷贝的类和原先的pixel都指向同一rgb对象，属于浅拷贝（shallow copy）。

  

**其他特征**

- 静态成员（变量/方法）,类的成员默认是动态的，可添加关键字static创建共享的静态成员变量或者方法
- 使用this来索引当前所在对象的成员（变量/参数/方法）
- 数据的隐藏，使得类的测试和维护变得简单。例如local，protected

## 类的继承

----

SV中通过使用关键词extends，可以扩展一个新的类继承于先前定义过的类，包括其所有的成员（变量/方法）。

**成员的覆盖**

子类可以继承父类的成员。当父类句柄指向子类的对象时，例如：

```verilog
LinkedPacket lp = new;
Packet p = lp; 
```

因为子类中声明了与父类同名的成员（变量/方法），其同名成员的访问都将指向子类，而父类的成员将被隐藏。

```verilog
class Packet;
  integer i = 1;
  function integer get();
    get = i;
  endfunction
endclass

class LinkedPacket extends Packet;
  integer i = 2;
  function integer get();
    get = -i;
  endfunction
endclass

LinkedPacket lp = new;
Packet p = lp;

j = p.i; // j = 1, not 2
j = p.get(); // j = 1, not -1 or –2

```

**super 的使用**

用于访问当前对象的父类成员，尤其当子类与父类的成员同名时，可以使用其指定访问父类成员，而非默认的子类成员。

## 包的使用

----

为了使得可以在多个模块（硬件）或者类（软件）之间共享用户定义的类型，SV添加了包（package）。

module、interface、class等可以使用包中定义或者声明的内容。

可以通过域的索引符::号直接引用,或者通过通配符*来将包中所有的类别导入到指定容器中。

----

*Reference*： 

1.路科验证V0系列课程，可参考《芯片验证漫游指南：从理论到UVM的验证全视界》