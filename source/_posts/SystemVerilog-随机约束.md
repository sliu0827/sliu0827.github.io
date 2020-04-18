---
title: SystemVerilog 随机约束
date: 2020-04-13 22:32:35
tags:
- SystemVerilog
categories:
 - 芯片验证基础
---



随着设计变得越来越大，要产生一个完整的激励来测试设计的功能也变得越来越困难。随机-约束，两个词组合在一起就构成了目前动态仿真验证的主流方法。

<!--more-->

随机约束测试（CRT，Constrained-Random Test）即能够产生你感兴趣的、你想不到的的测试向量，通过回归测试、替换随机种子的方式来提高单位测试用例的覆盖率收集效率。

面向DUT的随机激励产生过程中，为了符合协议、满足测试需求，我们需要添加一些“约束”，从而系统地组织随机变量。同时，采用类来作为载体来容纳这些变量和它们之间的约束，类的成员变量均可声明为“随机”属性，用rand或者randc来表示。

## 随机变量

-----

### 修饰符rand/randc

- rand修饰符，表示在可生成的范围内，每个值的可能性是相同的

  ```verilog
  rand bit [7:0] y;
  ```

- randc修饰符，其值会随机并且遍历其可取值范围。

  ```verilog
  randc bit [1:0] y;
  ```

  ![SV随机约束5.1.png](https://i.loli.net/2020/04/16/GMmdreoPKTt5N6u.png)

### 使用范围

- 任何类中的整形（bit/byte/int）变量都可以声明为rand/randc；
- 定长数组、动态数组、关联数组和队列都可以声明为rand/randc，可以对动态数组和队列的长度加以约束；
- 指向对象的句柄成员，也可以声明为rand(不能被声明为randc)，随机时该句柄指向对象中的随机变量也会一并被随机；
- 非组合型结构体可以声明为rand，非组合型的成员可以声明为rand/randc；

## 随机约束

-----

没有约束的随机变量会包含许多无效的和非法的值，这会使得有效激励的产生变得低效。因此，需要使用约束块来定义随机变量的约束关系。

### 约束块方法

- 成员集合，约束块支持整形通过set操作符来设置他们的可取值范围

  ```verilog
  rand integer x, y, z;
  constraint c1 {x inside {3, 5, [9:15], [24:32], [y:2*y], z};}
  
  rand integer a, b, c;
  constraint c2 {a inside {b, c};}
  
  integer fives[4] = '{ 5, 10, 15, 20 };
  rand integer v;
  constraint c3 { v inside {fives}; }
  
  ```

- 权重分布，可支持设置可取值的同时为其设置随机时的权重

  使用:=操作符，表示每个值的固定权重值；使用:/操作符，表示权重会平均分配到每一个值。

  ```verilog
  //x在100,101,102,200和300的权重是1-1-1-2-5。
  x dist { [100:102] := 1, 200 := 2, 300 := 5}
  
  //x在100,101,102,200和300的权重是1/3-1/3-1/3-2-5。
  x dist { [100:102] :/ 1, 200 := 2, 300 := 5}
  
  ```

- 唯一标识，unique可以用来约束一组变量，使其在随机后变量之间不会有相同的数值

- 条件约束，可以使用if-else或者->操作符来表示条件约束

  ```verilog
  constraint c1{
      mode == little -> len < 10;
      mode == big -> len > 100;}
  
  bit [3:0] a, b;
  constraint c2 { (a == 0) -> (b == 1); }
  
  constraint c3{
      if (mode == little)
          len < 10;
      else if (mode == big)
          len > 100;}
  ```

- 迭代约束，foreach可以用来迭代约束数组中的元素，这些数组可以是定长数组、动态数组、关联数组或者队列；

  ```verilog
  class C;
    rand byte A[] ;
    constraint C1 { foreach (A[i]) A[i] inside 
                    {2,4,8,16};}
    constraint C2 { foreach (A[j]) A[j] > 2 * j; }
  endclass
  ```

  对于某些数组，例如动态数组，也可以使用缩减的方法做迭代约束。

  ```verilog
  class C;
    rand bit [7:0] A[] ;
    constraint c1 { A.size() == 5; }
    constraint c2 { A.sum() < 1000; }
  endclass
  
  //A[0] + A[1] + A[2] + A[3] + A[4] < 1000
  ```

- 函数调用，可在约束块中调用函数来描述约束。例如，计算一个合并数组中的“1”的数量：

  ```verilog
  function int count_ones ( bit [9:0] w );
    for( count_ones = 0; w != 0; w = w >> 1 )
    count_ones += w & 1'b1;
  endfunction
  
  //可以在约束块中调用该函数来描述约束
  constraint C1 { length == count_ones( v ) ; }
  ```

- 软约束，没有soft描述时默认为硬约束，当约束冲突时，硬约束可以“覆盖”软约束，并且不会导致随机数产生的失败。

## 约束控制

----

### 随机方法randomize()

在类中声明的随机变量，需要伴随着**SV类的内建方法randomize()**的调用。

```verilog
virtual function int randomize();
```

如果随机化成功则返回1，如果失败则返回0。

```verilog
class SimpleSum;
  rand bit [7:0] x, y, z;
  constraint c {z == x + y;}
endclass
SimpleSum p = new;
int success = p.randomize();
if (success == 1 ) ...
```

**内嵌约束**：在调用类方法randomize()时可以伴随着with来添加额外的约束。

内嵌约束可能存在指向模糊的问题。例如以下：

```verilog
class C1;
  rand integer x;
endclass
class C2;
  integer x;
  integer y;
  task doit(C1 f, integer x, integer z);
    int result;
    result = f.randomize() with {x < y + z;};
  endtask
endclass
```

### 内嵌约束指向模糊问题

**解决指向模糊的问题，可以在随机时在with后面添加指定对象的成员变量**，不在其中的则是该对象以外的变量。

```verilog
class C;
  rand integer x;
endclass
function int F(C obj, integer y);
  F = obj.randomize() with (x) { x < y; };
endfunction

```

**可以通过local::的域索引方式来明确随机变量的指向**，即local::指向的变量会在包含randomize()方法的对象中。

```verilog
class C;
  rand integer x;
endclass
function int F(C obj, integer x);
  F = obj.randomize() with { x < local::x; };
endfunction
//第一个x指向对象c中的成员变量x，第二个x指向c对象以外的域
```

## 随机控制

-----

**随机变量的控制**

rand_mode可以用来使能或者禁止随机变量。

可以就单个随机变量调用其rand_mode，或者对整个对象调用rand_mode来控制其中所有的随机变量。

```verilog
task object[.random_variable]::rand_mode( bit on_off );
function int object.random_variable::rand_mode();
```

**约束块的控制**

通过约束控制函数constraint_mode来使能或者关闭某些约束块。

一些约束块或者某个类的约束块集合也可以单个控制或者集体控制。

```verilog
task object[.constraint_identifier]::constraint_mode(bit on_off );
function int object.constraint_identifier::constraint_mode();
```

**内嵌变量控制**

在使用类的随机化函数randomize()时，如果伴有参数，那么只会随机化这些变量，而其余变量无论是否之前被声明为rand/randc，都将不会参与到随机化当中。

```verilog
class CA;
  rand byte x, y;
  byte v, w;
  constraint c1 { x < v && y > w； };
endclass
CA a = new;
a.randomize(); // random variables: x, y 
a.randomize( x ); // random variables: x 
a.randomize( v, w ); // random variables: v, w
a.randomize( w, x ); // random variables: w, x 
```

----

*Reference*： 

1.路科验证V0系列课程，可参考《芯片验证漫游指南：从理论到UVM的验证全视界》

