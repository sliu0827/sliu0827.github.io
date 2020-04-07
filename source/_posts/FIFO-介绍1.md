---
title: FIFO 介绍1
date: 2020-03-14 23:10:01
tags:
 - 数字IC
categories:
 - 数字IC设计基础
---

FIFO的基本介绍，包括同步FIFO、异步FIFO、异步FIFO中常使用的格雷码（Gray code）计数器等

<!--more-->

## FIFO常见参数

-----

FIFO的宽度：即FIFO一次读写操作的数据位；

FIFO的深度：FIFO可以存储多少个N位的数据（如果宽度为N）；

满标志：FIFO已满或将要满时由FIFO的状态电路送出的一个信号，已阻止FIFO的写操作继续向FIFO中写数据而造成溢出（overflow）；

空标志：FIFO已空或将要空时由FIFO的状态电路送出的一个信号，以阻止FIFO的读操作继续从FIFO中读出数据而造成无效数据的读出（underflow）；

读时钟：读操作所遵循的时钟，在每个时钟沿来临时读数据；

写时钟：写操作所遵循的时钟，在每个时钟沿来临写数据。

## FIFO pointers

-----

正确的产生空满标志是任何FIFO设计的关键。**空满标志产生的原则是：写满而不溢出，能读空而不多读。**

**空满状态的判断**

当读写指针相等时，表明FIFO为空，这种情况发生在：

- 复位操作时
- 读指针读出FIFO中最后一个字后，追赶上了写指针

当读写指针再次相等时，表明FIFO为满，这种情况发生在：

- 写指针转了一圈，折回来又追上了读指针

### 同步FIFO指针(Synchronous FIFO pointers)：

同步FIFO中，在同一时钟域内进行FIFO缓冲区的读写操作。对FIFO缓冲区的写入和读取次数进行计数。

- increment：FIFO write but no read；

- decrement：FIFO read but no write;

- hold: no writes and reads,or simultaneous write and read operation

直接根据计数器数值来判断空满状态，当计数器达到预设的满值，FIFO填满。当计数器为0时，FIFO为空。

### 异步FIFO指针(Asynchronous FIFO pointers)：

异步FIFO设计中，则不能简单地增减FIFO填充指针，因为需要使用两个不同的异步时钟来控制计数器。因此，为了确定异步FIFO设计的满状态和空状态，必须比较写指针和读指针。

**当读指针等于写指针时，FIFO为空。**该情况发生在所有指针复位期间复位为0时，或者当读指针赶上写指针，读到FIFO最后一个字节的情况下。

**当指针再次相等，即写指针循环一次后再次追赶上读指针，FIFO为满。**

为了区分满空状态，我们可以为每个指针添加一个额外的位MSB。写指针增加到FIFO最后一个地址后，将未使用的MSB递增，同时将其余位设置为0，如下图所示。读指针也是如此。

**如果两个指针的MSB不同，则意味着写指针比读指针多循环了一次。如果两个指针的MSB相同，则意味着两个指针的换行次数相同。**

![FIFO3.2.png](https://i.loli.net/2020/03/26/WJ5OszSy1GmMtYn.png)

## FIFO结构实例：

-----

下图为典型的FIFO结构实例，主要包括：

FIFO memory：存储缓存区，可由写入和读取时钟域访问。Memory buffer大多由synchronous dual-port RAM或其它memory 例化而来；

sync_r2w&sync_w2r:同步器电路，用于将读指针同步到写时钟域或将写指针同步到读时钟域中。wptr_full使用同步的读指针来生成满条件；rptr_empty使用同步的写指针来生成FIFO空条件；

rptr_empty：与读时钟域完全同步，并包含了FIFO读指针和空标志逻辑；

wptr_full:与写时钟域完全同步，并包含了FIFO写指针和满标志逻辑。

![FIFO3.1.png](https://i.loli.net/2020/03/26/ynJBlsjCx6UTmv2.png)

为了实现FIFO full、empty的判断，读写指针需要传递到对方时钟域中进行比较。为了保证安全的传输到对方时钟域，后面会介绍同步格雷码指针的方法，保证一次只更改一个指针位。

## 使用格雷码（Gray Code）计数器

-----

### 格雷码的特点：

- 相邻的2个数值之间只会有1位发生变化，其余各位相同；
- 是一种循环码，0和最大数（2的n次方减1）之间也只有一位不同

### Gray To Binary Conversion

4 bit Gray To Binary Conversion:

```verilog
bin[0]= gray[3] ^ gray[2] ^ gray[1] ^ gray[0];
bin[1] = gray[3] ^ gray[2] ^ gray[1];
bin[2] = gray[3] ^ gray[2];
bin[3] = gray[3];
```

verilog 实现，通过采用填充0 的方式对有效的格雷码位进行异或操作，如下所示：

```verilog
bin[0] = gray[3] ^ gray[2] ^ gray[1] ^ gray[0] ; // gray>>0
bin[1] =   1'b0  ^ gray[3] ^ gray[2] ^ gray[1] ; // gray>>1
bin[2] =   1'b0  ^   1'b0  ^ gray[3] ^ gray[2] ; // gray>>2
bin[3] =   1'b0  ^   1'b0  ^   1'b0  ^ gray[3] ; // gray>>3
```

Code:

```verilog
module gray2bin (bin, gray);
  parameter SIZE = 4;
  output [SIZE-1:0] bin;
  input  [SIZE-1:0] gray;
  reg    [SIZE-1:0] bin;
  integer           i;
  always @(gray)
    for (i=0; i<SIZE; i=i+1)
      bin[i] = ^(gray>>i);
endmodule
```

### Binary To Gray Conversion

4bit binary-to-gray conversion:

```verilog
gray[0] = bin[0] ^ bin[1];
gray[1] = bin[1] ^ bin[2];
gray[2] = bin[2] ^ bin[3];
gray[3] = bin[3];
```

Code:

```verilog
module bin2gray (gray, bin);
  parameter SIZE = 4;
  output [SIZE-1:0] gray;
  input  [SIZE-1:0] bin;
  assign gray = (bin>>1) ^ bin;
endmodule
```

### Gray Code Counter

verilog Code中包含一个gray-to-binary 和一个binary-to-gray转换器，并在两者之间递增1。

```verilog
module graycntr (gray, clk, inc, rst_n);
  parameter SIZE = 4;
  output [SIZE-1:0] gray;
  input             clk, inc, rst_n;
  reg    [SIZE-1:0] gnext, gray, bnext, bin;
  integer           i;
  always @(posedge clk or negedge rst_n)
    if (!rst_n) gray <= 0;
    else        gray <= gnext;
  always @(gray or inc) begin
    for (i=0; i<SIZE; i=i+1)
      bin[i] = ^(gray>>i);
    bnext = bin + inc;
    gnext = (bnext>>1) ^ bnext;
  end
endmodule
```

Gray-code counter block diagram：

![FIFO3.3.png](https://i.loli.net/2020/03/26/yisngNOo1lAqkTu.png)

## 使用Gray Code后，如何做空满判断？

-----

空状态：依据二者完全相等（包括MSB）

满状态：由于gray码除了MSB外，具有镜像对称的特点，存在一个特殊情况。当读指针指向7，写指针指向8时，除了MSB，其余位皆相同，不能算满。

![FIFO3.4.png](https://i.loli.net/2020/03/26/ew4r8F5H6muf9Pp.png)

未解决上述出现的问题，特地改进Gray code counter，如下图所示,采用一种dual n-bit Gray code counter。

*n-bit Gray code converted to an (n-1)-bit Gray code*

![FIFO3.5.png](https://i.loli.net/2020/03/26/br8mwJD6tAKVTN5.png)

*Dual n-bit Gray code counter block diagram*

![FIFO3.6.png](https://i.loli.net/2020/03/26/tAyQmYOBGlVFg9b.png)

使用Grey判断须同时满足以下3条：

- wptr和同步过来的rptr的**MSB不相等**；
- wptr与rptr的**次高位不相等**，如上图位置7和位置15，转化为二进制对应的是0111和1111，MSB不同说明多折回一次，111相同代表同一位置；
- **剩下的其余位完全相等**。

## RTL code for FIFO 

-----

### fifo.v-FIFO top-level module

```verilog
module fifo1 #(parameter DSIZE = 8,
               parameter ASIZE = 4)
  (output [DSIZE-1:0] rdata,
   output             wfull,
   output             rempty,
   input  [DSIZE-1:0] wdata,
   input              winc, wclk, wrst_n,
   input              rinc, rclk, rrst_n);
  wire   [ASIZE-1:0] waddr, raddr;
  wire   [ASIZE:0]   wptr, rptr, wq2_rptr, rq2_wptr;
  sync_r2w      sync_r2w  (.wq2_rptr(wq2_rptr), .rptr(rptr),
                           .wclk(wclk), .wrst_n(wrst_n));
  sync_w2r      sync_w2r  (.rq2_wptr(rq2_wptr), .wptr(wptr),
                           .rclk(rclk), .rrst_n(rrst_n));
  fifomem #(DSIZE, ASIZE) fifomem
                          (.rdata(rdata), .wdata(wdata),
                           .waddr(waddr), .raddr(raddr),
                           .wclken(winc), .wfull(wfull),
                           .wclk(wclk));
  rptr_empty #(ASIZE)     rptr_empty
                          (.rempty(rempty),
                           .raddr(raddr),
                           .rptr(rptr), .rq2_wptr(rq2_wptr),
                           .rinc(rinc), .rclk(rclk),
                           .rrst_n(rrst_n));
  wptr_full  #(ASIZE)     wptr_full
                          (.wfull(wfull), .waddr(waddr),
                           .wptr(wptr), .wq2_rptr(wq2_rptr),
                           .winc(winc), .wclk(wclk),
                           .wrst_n(wrst_n));
endmodule
```

### fifomem.v - FIFO memory buffer

```verilog
module fifomem #(parameter  DATASIZE = 8, // Memory data word width
                 parameter  ADDRSIZE = 4) // Number of mem address bits
  (output [DATASIZE-1:0] rdata,
   input  [DATASIZE-1:0] wdata,
   input  [ADDRSIZE-1:0] waddr, raddr,
   input                 wclken, wfull, wclk);
  ìfdef VENDORRAM
    // instantiation of a vendor's dual-port RAM
    vendor_ram mem (.dout(rdata), .din(wdata),
                    .waddr(waddr), .raddr(raddr),
                    .wclken(wclken),
                    .wclken_n(wfull), .clk(wclk));
  èlse
    // RTL Verilog memory model
    localparam DEPTH = 1<<ADDRSIZE;
    reg [DATASIZE-1:0] mem [0:DEPTH-1];
    assign rdata = mem[raddr];
    always @(posedge wclk)
      if (wclken && !wfull) mem[waddr] <= wdata;
  èndif
endmodule
```

### sync_r2w.v - Read-domain to write-domain synchronizer

```verilog
module sync_r2w #(parameter ADDRSIZE = 4)
  (output reg [ADDRSIZE:0] wq2_rptr,
   input      [ADDRSIZE:0] rptr,
   input                   wclk, wrst_n);
  reg [ADDRSIZE:0] wq1_rptr;
  always @(posedge wclk or negedge wrst_n)
    if (!wrst_n) {wq2_rptr,wq1_rptr} <= 0;
    else         {wq2_rptr,wq1_rptr} <= {wq1_rptr,rptr};
endmodule
```

### sync_w2r.v - Write-domain to read-domain synchronizer

```verilog
module sync_w2r #(parameter ADDRSIZE = 4)
  (output reg [ADDRSIZE:0] rq2_wptr,
   input      [ADDRSIZE:0] wptr,
   input                   rclk, rrst_n);
  reg [ADDRSIZE:0] rq1_wptr;
  always @(posedge rclk or negedge rrst_n)
    if (!rrst_n) {rq2_wptr,rq1_wptr} <= 0;
    else         {rq2_wptr,rq1_wptr} <= {rq1_wptr,wptr};
endmodule
```

### rptr_empty.v - Read pointer & empty generation logic

```verilog
module rptr_empty #(parameter ADDRSIZE = 4)
  (output reg                rempty,
   output     [ADDRSIZE-1:0] raddr,
   output reg [ADDRSIZE  :0] rptr,
   input      [ADDRSIZE  :0] rq2_wptr,
   input                     rinc, rclk, rrst_n);
  reg  [ADDRSIZE:0] rbin;
  wire [ADDRSIZE:0] rgraynext, rbinnext;
  //-------------------
  // GRAYSTYLE2 pointer
  //-------------------
  always @(posedge rclk or negedge rrst_n)
    if (!rrst_n) {rbin, rptr} <= 0;
    else         {rbin, rptr} <= {rbinnext, rgraynext};
  // Memory read-address pointer (okay to use binary to address memory)
  assign raddr     = rbin[ADDRSIZE-1:0];
  assign rbinnext  = rbin + (rinc & ~rempty);
  assign rgraynext = (rbinnext>>1) ^ rbinnext;
  //---------------------------------------------------------------
  // FIFO empty when the next rptr == synchronized wptr or on reset
  //---------------------------------------------------------------
  assign rempty_val = (rgraynext == rq2_wptr);
  always @(posedge rclk or negedge rrst_n)
    if (!rrst_n) rempty <= 1'b1;
    else         rempty <= rempty_val;
endmodule
```

### wptr_full.v - Write pointer & full generation logic

```verilog
module wptr_full  #(parameter ADDRSIZE = 4)
  (output reg                wfull,
   output     [ADDRSIZE-1:0] waddr,
   output reg [ADDRSIZE  :0] wptr,
   input      [ADDRSIZE  :0] wq2_rptr,
   input                     winc, wclk, wrst_n);
  reg  [ADDRSIZE:0] wbin;
  wire [ADDRSIZE:0] wgraynext, wbinnext;
  // GRAYSTYLE2 pointer
  always @(posedge wclk or negedge wrst_n)
    if (!wrst_n) {wbin, wptr} <= 0;
    else         {wbin, wptr} <= {wbinnext, wgraynext};
  // Memory write-address pointer (okay to use binary to address memory)
  assign waddr = wbin[ADDRSIZE-1:0];
  assign wbinnext  = wbin + (winc & ~wfull);
  assign wgraynext = (wbinnext>>1) ^ wbinnext;
  //------------------------------------------------------------------
  // Simplified version of the three necessary full-tests:
  // assign wfull_val=((wgnext[ADDRSIZE]    !=wq2_rptr[ADDRSIZE]  ) &&
  //                   (wgnext[ADDRSIZE-1]  !=wq2_rptr[ADDRSIZE-1]) &&
  //                   (wgnext[ADDRSIZE-2:0]==wq2_rptr[ADDRSIZE-2:0]));
  //------------------------------------------------------------------
  assign wfull_val = (wgraynext=={~wq2_rptr[ADDRSIZE:ADDRSIZE-1],
                                   wq2_rptr[ADDRSIZE-2:0]});
  always @(posedge wclk or negedge wrst_n)
    if (!wrst_n) wfull  <= 1'b0;
    else         wfull  <= wfull_val;
endmodule
```



------

*Reference*： 

1.[Simulation and Synthesis Techniques for Asynchronous FIFO Design](http://www.sunburst-design.com/papers/CummingsSNUG2002SJ_FIFO1.pdf)

2.[芯动力—硬件加速设计方法](https://www.icourse163.org/course/SWJTU-1207492806?tid=1207824209)