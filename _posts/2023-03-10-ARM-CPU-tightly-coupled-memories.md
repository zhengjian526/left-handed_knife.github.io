---
layout: post
title: ARM的紧耦合内存--TCM
categories: 硬件相关
description: ARM的紧耦合内存--TCM
keywords: ARM, TCM, CPU
---

# TCM简介
TCM ：TIghtly Coupled Memory 的缩写。为了弥补 Cache 访问的不确定性，而增加的 OnChip Memory. 有的 CPU 含有分立的 InstrucTIon TCM / Data TCM.TCM 包含在存储器的地址映射空间中，可以作为快速存储器来访问。TCM 使用物理地址，对 TCM 的写访问，受到 MMU 内部保护信息的控制。向 TCM 中的内存位置写入时，不会发生任何外部写入。

TCM 用于向处理器提供低延迟内存，它没有高速缓存特有的不可预测性。 可以使用 TCM 来存放重要例程，如中断处理例程或者极需要避免高速缓存不确定性的实时任务。此外，可以使用 TCM 来保存暂时[寄存器](https://www.eefocus.com/baike/502591.html)数据、局部属性不适合高速缓存的数据类型，以及中断堆栈等重要数据结构。

[ARM](https://www.eefocus.com/baike/481548.html) 的 ram 包括静态 ram，动态 ram， TCM--- 紧耦合内存（TCM： TIghtly Coup ledMemories）。

TCM 是一个固定大小的 RAM，紧密地耦合至处理器内核，提供与 cache 相当的性能，相比于 cache 的优点是，程序代码可以精确地控制什么函数或代码放在哪儿（RAM 里）。当然 TCM 永远不会被踢出主存储器，因此，他会有一个被用户预设的性能，而不是象 cache 那样是统计特性的性能提高。

## **紧致内存介绍**

紧致内存是指片上快速存储区，与片上缓存具有同等的性能，但因为程序可完全控制紧致内存，因而比统计复用的缓存有更好的可预测性。这是 ARM5TE 引入的特性，目的是通过这一快速的存储区，一方面提高某些关键代码（如中断处理函数）的性能，另方面使存储访问延迟保持一致，这是实时性应用所要求的。ARM6 对 TCM 操作做了进一步的规范。

## TCM应用领域

可预测的实时处理（**中断处理**）、避免缓存分析（加密算法）、或单纯的性能提高（处理器侧编解码）等。

如同缓存的哈佛结构，指令 TCM 和数据 TCM 是分开的。TCM 有两种使用方式：作为快缓存使用，和作为本地内存使用。



# R5F的TCM基本信息介绍

由于工作涉及到R5F子系统的相关设计实现，所以本文的相关介绍以R5F为例，具体信息以[官网技术手册为准](https://developer.arm.com/documentation/ddi0460/latest/)。

TCM是在集成实现过程中可配置的接口，处理器有两个TCM接口来支持本地内存的连接。**ATCM的接口上有1个TCM端口。BTCM接口支持1个或2个TCM端口**。每一个TCM端口是处理器上的一个物理连接，适合连接到SRAM最小的胶水逻辑。这些端口针对低延迟内存进行了优化。

**TCM的接口设计为连接RAM，即类RAM内存，即普通类型内存。**

处理器可以在这些接口上发出推测的读访问，并中断已经发出部分但不是全部写访问的存储指令。因此，通过TCM接口的读写访问都可以重复进行。这意味着TCM端口通常不适合读或写敏感的设备，例如fifo。ROM可以连接到TCM端口，但通常只有在不使用ECC的情况下。

PFU可以通过TCM接口读取数据。LSU和AXI从机都可以通过TCM接口读写数据。

**每个TCM接口都有一个专用的基址，可以将其放置在物理地址映射中的任何位置，并且不能由外部实现的内存提供支持。ATCM接口和BTCM接口必须有独立的基址，且不能重叠。**

可以配置移除ATCM接口，不包含在处理器的设计当中。如果配置实现了ATCM接口，ATCM只能有一个端口。

BTCM的接口可配置项包括：

- 被移除，不包含在处理器设计中
- 拥有单个BTCM端口
- 有两个存储BTCM端口，交错使用: 地址的[3]位， BTCM接口地址中最重要的位。这取决于的大小BTCM。

在实现过程中，您可以配置ATCM和/或BTCM使用错误保护方案，用于保护TCM中存储的数据，请参见TCM内部错误检测和修正。

每个TCM接口的大小在集成时配置。允许的TCM尺寸是:

- 0 kb
- 4 kb
- 8 kb
- 16 kb
- 32 kb
- 64 kb
- 128 kb
- 256 kb
- 512 kb
- 1 mb
- 2 mb
- 4 mb
- 8 mb。

如果BTCM接口有两个端口，则每个端口所附加的RAM大小为BTCM接口总大小的一半。

## 其他

目前仅是使用接口进行读写消息数据，实际具体遇到问题再具体分析记录。

## 参考

- https://www.eefocus.com/article/398765.html
- R5F技术参考手册：https://developer.arm.com/documentation/ddi0460/latest/