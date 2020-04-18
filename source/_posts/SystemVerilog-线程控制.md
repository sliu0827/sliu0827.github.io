---
title: SystemVerilog 线程控制
date: 2020-04-17 11:41:51
tags:
 - SystemVerilog
categories:
 - 芯片验证基础
---



介绍SV中的任务和函数，线程控制和线程间的同步和通信的基本概念

<!--more-->

## 任务和函数

-----

### 函数（function）

首要目的在于为运算表达式提供返回值，其次可以简化代码，使得大型代码的维护更加容易。

函数的参数列表主要有input、output、inout和ref。

**函数的返回方式包括return直接返回，或者将值赋给与函数同名的变量。**

```verilog
function [15:0] myfunc1 (input [7:0] x,y);
  myfunc1 = x * y - 1; 
endfunction
function [15:0] myfunc2 (input [7:0] x,y);
  return x * y - 1; 
endfunction
```

如果调用具有返回值的函数，但是又不使用该返回值，可以添加void'()进行转换。

```verilog
void'(some_function());
```

### 任务（task）

任务的定义同样可以指定函数中包括的各种参数，但任务可以消耗仿真时间。同时，任务可以调用其它任务或者函数。

task没有返回值，但我们依然可以利用return来使task结束。

### 二者区别

- function不会消耗仿真时间，而task则可能会消耗仿真时间；
- function无法调用task，而task可以调用function；
- 一个可以返回数据的function只能返回一个单一数值，而任务或者void function不会返回数值；
- 一个可以返回数据的function可以作为一个表达式中的操作数，该操作数的值即function的返回值。

### 参数传递

参数传递的过程只发生在方法的调用和返回时。

**ref参数在传递时不会发生值拷贝，而是将变量“指针”传递到方法中，在方法内部对该参数的操作将会同时影响外部变量。**

如果为了避免外部传入的ref参数会被方法修改，则可以添加const修饰符，来表示变量是只读变量。

SV中允许参数声明自己的默认值。方法被调用时，可由参数位置顺序在调用方法时传递参数，也可以由参数名字调用方式时绑定参数。

## 线程控制

----

**并行线程**

Verilog中与顺序线程begin...end相对的是并行线程fork...join，SV引入了两种新的创建线程的方法，fork...join_none和fork...join_any。

![SV线程控制6.1.png](https://i.loli.net/2020/04/17/a3kyPvwTEzx7KjQ.png)

fork...join需要所有并行的线程都结束以后才会继续执行;

fork...join_any则会等到任何一个线程结束以后就继续执行;

fork...join_none则不会等待其子线程而继续执行。

对于并行线程中，例如fork...join_any和fork...join_none继续执行后，未完成的后台子程序，可以通过使用wait fork或disable fork来等待或者终止这些子程序。

**时序控制**

SV可以通过延迟控制或者事件等待来对过程块完成时序控制。

- 延迟控制即通过#来完成

  ```veril
  #10 rega = regb;
  ```

- 时间（event）控制即通过@来完成

  ```verilog
  @r rega = regb;
  @(posedge clock) rega = regb;
  ```

- wait语句也可以与事件或者表达式结合来完成

  ```verilog
  real AOR[];
  initial wait(AOR.size() > 0)...;
  ```

## 线程的同步和通信

----

 测试平台中的所有线程都需要同步并交换数据。例如，一个线程等待另外一个，验证环境需要等待所有激励结束、比较结束才可以结束仿真。又有，监测器需要将监测到的数据发送至比较器，比较器又需要从不同的缓存获取数据进行比较。

### 事件（event）

event可以用来控制进程的同步。

一般通过使用event来声明event变量，通过->来触发事件。其他等待该事件的进程可以通过@操作符或者wait()来检查event触发状态来完成。

```verilog
event done, blast; // declare two new events
event done_too = done; // done_too as alias to done
task trigger( event ev );
  -> ev;
endtask
...

fork
  @ done_too; // wait for done through done_too
  #1 trigger( done ); // trigger done
join

fork
  -> blast;
  wait ( blast.triggered );
join
```

### wait_order()方法

对于较多的事件时，wait_order可以使得进程保持等待，直到在参数列表中的事件event按照顺序从左到右依次完成。

```verilog
wait_order( a, b, c);

wait_order( a, b, c ) else $display( "Error: events out of order" );

bit success;
wait_order( a, b, c ) success = 1; else success = 0;
```

### 旗语（semaphore）

多个线程对同一资源进行访问时，使用旗语来实现对同一资源的访问控制，也称为“互斥访问”。其可以确保线程A在访问的时候，线程B是无法访问的，A退出后，B才可以访问。

旗语的基本操作包括：

- new方法创建一个带单个或多个钥匙的旗语；
- get获取一个或多个钥匙；
- put则可以返回一个或多个钥匙

如果试图获取一个旗语而不被阻塞，则可以使用try_get()函数，返回1则表示有足够的钥匙。

如果一个线程请求“钥匙”不足，则会一直阻塞，多个阻塞的线程会以先进先出（FIFO）的方式进行排队。

### 信箱（mailbox）

信箱mailbox可以使得进程之间的信息得以交换，数据可以由一个进程写入信箱，再由另外一个进程获得。

**信箱的内建方法：**

- 创建信箱：new()
- 将信息写入信箱：put()，试着写入信箱但不会阻塞：try_put()
- 获取信息：get()同时会取出数据，peek()不会取出数据
- 试着从信箱取出数据但不会阻塞：try_get()/try_peek()
- 获取信箱信息的数目：num()

信箱的参数设置，没有指定参数时可以存放任何类型的数据，但一般不推荐这种操作。参数化信箱的方式可以使得在编译时就能够检查出类型不匹配的情况。

```verilog
typedef mailbox #(string) s_mbox;
s_mbox sm = new;
string s;
sm.put( "hello" );
...
sm.get( s ); // s <- "hello"
```

----

*Reference*： 

1.路科验证V0系列课程，可参考《芯片验证漫游指南：从理论到UVM的验证全视界》

