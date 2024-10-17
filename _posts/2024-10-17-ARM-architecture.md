---
layout: post
title: ARM架构基础
date: 2024-10-17
author: 云萧雨霁
tags: [ARM]
comments: true
toc: true
pinned: true
---

[3. ARMv8 基础知识 — Armv8/armv9架构入门指南 v1.0 文档](http://hehezhou.cn/arm/03.html)

# 内存排序（Memory Ordering）：
参考文档：[13. 内存排序 — Armv8/armv9架构入门指南 v1.0 文档](http://hehezhou.cn/arm/13.html)

## 内存类型：
<font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">ARM v8 v7 架构定义了两种互斥的内存类型。所有内存区域都配置为这两种类型中的一种，即Normal和Device，以及 Strongly Ordered。</font>

![节选自《Zynq-7000-TRM》](./pic/2024-10-17-ARM-architecture/L (1).png)

![](https://cdn.nlark.com/yuque/0/2024/png/21562848/1720422526747-6e7c3210-4262-4210-8571-553b06519292.png)

+ <font style="color:rgb(0, 0, 0);background-color:rgb(238, 238, 238);">1、对 Strongly-ordered 存储器的写入，只有在写访问到外设或者存储器组件的时候，才算完成；</font>
+ <font style="color:rgb(0, 0, 0);background-color:rgb(238, 238, 238);">2、对 Device 存储器的写入，允许在写访问到达外设之前就完成；</font>

## 内存属性：
[https://juejin.cn/post/7354045058265202715](https://juejin.cn/post/7354045058265202715)

<font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">内存属性提供对</font>**<font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">可缓存性</font>**<font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">、</font>**<font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">可共享性</font>**<font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">、</font>**<font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">访问</font>**<font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">和</font>**<font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">执行权限</font>**<font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">的控制。</font>

+ <font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">可共享仅适用于 </font>**<font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">normal memory</font>**
+ <font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">可缓存仅适用于 </font>**<font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">normal memory</font>**
+ <font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">设备区域始终被视为不可缓存且可外部共享。</font>
+ <font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">对于可缓存位置，您可以使用属性向处理器指示缓存分配策略。</font>

### <font style="color:rgb(64, 64, 64);background-color:rgb(252, 252, 252);">可共享：</font>
对于不同的 CPU 架构 cache 的属性可能不同，ZYNQ7000 配置为 L1 为 inner cache ，L2 为 outher cache。![ZYNQ7000](https://cdn.nlark.com/yuque/0/2024/png/21562848/1720524606316-f309f08f-e4c8-4cab-9480-8b97565a7f9a.png)

![可共享的几种属性选择-节选自《Zynq-7000-TRM》](https://cdn.nlark.com/yuque/0/2024/png/21562848/1720409392618-b3d4a0f9-8719-4180-8807-fab2aa0742b7.png)

![共享范围 - 《ARMv8-A memory systems》](https://cdn.nlark.com/yuque/0/2024/png/21562848/1720409717186-6338f17f-3ffc-4b54-bcb0-1929e213ab81.png)

![](https://cdn.nlark.com/yuque/0/2024/png/21562848/1720422582124-9ab60253-bd12-4fdb-944a-a6dd6b6bcc83.png)

![](https://cdn.nlark.com/yuque/0/2024/png/21562848/1720422588461-7a1fdda4-f8b5-47f7-8335-5074dbdeb118.png)

Device 类型存储器地址空间，不管是配置为 Non-Shareable，还是 Inner-Shareable，还是 Outer-Shareable，都一概视为 Outer-Shareable（毕竟是外设）；

### 可缓存
Normal 类型的存储器（比如 DDR）除了有 Shareable 的属性以外，还可以携带 cacheable 的属性：

+ Write-Through Cacheable：写透型 Cache 属性；
+ Write-Back Cacheable：回写型 Cache 属性；
+ Non-cacheable：不带 Cache 属性；

Non-Cacheable 很好理解，就是不带 Cache，直接与存储器交互，这样不会带来内存一致性问题，但是效率不高；

其余两种都是带 Cache 的访问，不过 Cache 的策略略有区别；写透也可以在一定程度上保证内存一致性问题，但要发起内存访问时序，降低性能；回写只是将数据写到了 Cache 中，并通过 Dirty 标记等方式来记录数据的有效性，从而避免直接的内存访问，可以提高性能；

![zynq 7000中的描述](https://cdn.nlark.com/yuque/0/2024/png/21562848/1728352577405-04220191-93c1-4295-aa2b-21c46fd1a6df.png)

<font style="color:rgb(42, 47, 69);">MMU 可以通过在 Translation Table Base 寄存器中设置 IRGN 位来配置硬件翻译表遍历功能，以在可缓存区域中执行。如果 IRGN 位的编码为写回，则会执行 L1 数据缓存查找，从数据缓存中读取数据。如果 IRGN 位的编码为写入或非缓存，则会执行外部内存访问。</font>

### 总结：
inner shareable domain 的影响范围主要由 cluster 决定，具体 cluster 大小参考架构的手册，例如：

![《A9 MPCore TRM手册》](https://cdn.nlark.com/yuque/0/2024/png/21562848/1720575850881-74be68c6-1bb7-43bb-a37d-bd563e44a3bc.png)

![ZYQN7000 共享类型影响范围理解图](https://cdn.nlark.com/yuque/0/2024/png/21562848/1720580532003-7a6fb5d5-a6f7-4c8e-a1de-0177298333f0.png)

:::success
**网友举的一个荔枝：**

![ARM v8](https://cdn.nlark.com/yuque/0/2024/webp/21562848/1720515212291-c119f3ac-b77e-4c8b-841b-0558255d637f.webp)

+ **如果将block的内存属性配置成Non-cacheable**，那么数据就不会被缓存到cache，那么所有observer看到的内存是一致的，也就说此时也相当于Outer Shareable。 其实官方文档，也有这一句的描述： 在B2.7.2章节 “Data accesses to memory locations are coherent for all observers in the system, and correspondingly are treated as being Outer Shareable”
+ **如果将block的内存属性配置成write-through cacheable 或 write-back cacheable**，那么数据会被缓存cache中。write-through和write-back是缓存策略。
+ **如果将block的内存属性配置成 non-shareable**, 那么core0访问该内存时，数据缓存的到Core0的L1 d-cache 和 cluster0的L2 cache，不会缓存到其它cache中
+ **如果将block的内存属性配置成 inner-shareable**, 那么core0访问该内存时，数据只会缓存到core 0和core 1的L1 d-cache中, 也会缓存到clustor0的L2 cache，不会缓存到clustor1中的任何cache里。
+ **如果将block的内存属性配置成 outer-shareable**, 那么core0访问该内存时，数据会缓存到所有cache中

:::

| | Non-cacheable | write-through   cacheable | write-back   cacheable |
| --- | --- | --- | --- |
| **non-shareable** | 数据不会缓存到cache   （对于观察则而言，又相当于outer-shareable） | Core0读取时，数据缓存的到Core0的L1 d-cache 和 cluster0的L2 cache, 如果core0和core1都读写过该内存，且在core0 core1的L1 d-cache中都缓存了该内存。那么core0在读取数据的时候，core0的L1 Dcache会更新，但core 1的L1 Dcache不会更新 | 同左侧 |
| **inner-shareable** | 数据不会缓存到cache   （对于观察则而言，又相当于outer-shareable） | Core0读取时，数据缓存的到Cluster0中所有cache | 同左侧 |
| **outer-shareable** | 数据不会缓存到cache   （对于观察则而言，又相当于outer-shareable） | Core0读取时，数据缓存的到所有cache | 同左侧 |


# 内存管理MMU：
![](https://cdn.nlark.com/yuque/0/2024/png/21562848/1720524896533-c82ae7e0-e964-420f-b8ad-5ec02201a61d.png)

# Cache
## cache 基本概念：
### 缓存行：
内存刷新 cache 都会最少将1Line 的数据刷新到 cache 中。

### Lcache 和 Dcache 的区别
:::info
L1 数据缓存 (L1D Cache)

+ **用途**：专门用于存储和缓存处理器在执行过程中需要的数据。
+ **访问速度**：非常快，通常是处理器能够直接访问的最快存储区域之一。
+ **大小**：通常较小，容量一般在16KB到64KB之间，具体取决于处理器架构。
+ **设计**：优化用于频繁访问的数据，例如局部变量、数组等。
+ **示例**：在执行诸如 `MOV` 或 `ADD` 等操作时，处理器会首先尝试从 L1 数据缓存中读取或写入数据。

L1 指令缓存 (L1I Cache)

+ **用途**：专门用于存储和缓存处理器即将执行的指令。
+ **访问速度**：与 L1 数据缓存一样快，确保指令可以快速提取以供执行。
+ **大小**：通常与 L1 数据缓存相当，容量一般在16KB到64KB之间。
+ **设计**：优化用于快速提取和预取指令，减少指令获取阶段的瓶颈。
+ **示例**：在执行程序时，处理器会从 L1 指令缓存中读取即将执行的指令。

:::

## 管理 Cache
内核中，对 Cache 的管理基本都在 `cp15 `协处理器中。

![《Arm A9 TRM》:cp15 SCTLR](https://cdn.nlark.com/yuque/0/2024/png/21562848/1721898626860-647eedc4-ef37-43e5-a5c7-1c7cef824a34.png)

## 多核缓存一致性问题
+ 如果要将定义在这里面的buffer作为DMA的源，用户必须在DMA开启之前执行一个D-Cache的清除操作，即SCB_CleanDCache()；如果该buffer用作DMA的目标，在DMA完成后且CPU或其它主机读取数据之前，需要执行SCB_InvalidateDCache。
+ buffer的地址应该基于L1缓存行的大小进行对齐，在Cortex-M7中为32字节

# 中断异常 GIC
## 多核GIC内存映射分区
![《ARM GIC v1》3.1](https://cdn.nlark.com/yuque/0/2024/png/21562848/1720606444819-f8a86a4e-882f-4927-9e3e-23d4633d935c.png)

 总结就是分配器相关的是cpu共用一个内存映射分区，其他的寄存器 cpu在访问时内存映射到不同位置了。





