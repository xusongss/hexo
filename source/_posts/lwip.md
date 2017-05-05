---
title: LwIP
date: 2017-05-05 14:54:27
tags:
photos:
- /doc/lwip/Design_of_LwIP_Integration.png
---
# LwIP 协议设计与实现

LwIP是瑞典计算机科学院(SICS)的Adam Dunkels开发的一个小型开源的TCP/IP协议栈。

LwIP是Light Weight (轻型)IP协议，有无作业系统的支持都可以运行。LwIP实现的重点是在保持TCP协议主要功能的基础上减少对RAM的占用，它只需十几KB的RAM和40K左右的ROM就可以运行，这使LwIP协议栈适合在低端的嵌入式系统中使用。[1]

LwIP协议栈主要关注的是怎么样减少内存的使用和代码的大小，这样就可以让lwIP适用于资源有限的小型平台例如嵌入式系统。为了简化处理过程和内存要求，lwIP对API进行了裁减，可以不需要复制一些数据。

LwIP由几个模块组成，除TCP/IP协议的实现模块外（ IP， ICMP， UDP， TCP），还有包括许多相关支持模块。这些支持模块包括：作业系统模拟层、缓冲与内存管理子系统、网络接口函数及一组Internet校验和计算函数。

[LwIP 协议设计与实现 中文版](/doc/lwip/Lwip-tw.pdf)
