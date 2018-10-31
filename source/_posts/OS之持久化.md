---
title: OS之持久化
date: 2018-08-13 10:01:38
tags:
    - 操作系统
---

问: IO设备是如何与操作系统进行通信的?

>  答: 通过总线

问: 什么是总线?

> 答: In [computer architecture](https://en.wikipedia.org/wiki/Computer_architecture), a **bus**[[1\]](https://en.wikipedia.org/wiki/Bus_(computing)#cite_note-1) (a contraction of the Latin *omnibus*) is a communication system that transfers data between components inside a [computer](https://en.wikipedia.org/wiki/Computer), or between computers. This expression covers all related hardware components (wire, optical fiber, etc.) and software, including communication protocols.[[2\]](https://en.wikipedia.org/wiki/Bus_(computing)#cite_note-2) 

问: 为什么要设计多级总线架构 ?

> 答: 多级总线针对的是不同IO设备所作出的优化, 低速设备连接到低速总线, 高速设备连接到高速总线
>
> ![](http://liimg.oss-cn-shenzhen.aliyuncs.com/18-8-13/59736801.jpg)



问: 一个IO设备的组成是什么?

>  答: 外部接口以及内部实现.
>
> ![](http://liimg.oss-cn-shenzhen.aliyuncs.com/18-8-13/29166608.jpg)

问: 操作系统如何与IO设备交互?

> 答: 
>
> | 方法 | 详细                                            | 缺点                            |
> | ---- | ----------------------------------------------- | ------------------------------- |
> | PIO  | 轮询IO设备的状态并给command和data寄存器写入数据 | 性能问题                        |
> | 中断 | 把PIO中轮询改为中断,IO设备把状态推给OS          | 中断处理, 上下文切换, CPU读写IO |
> | DMA  | CPU不再读写IO, DMA直接把IO设备中的数据放到内存  |                                 |
>
> 

问: 什么是RAID ?

> 答:  At a high level, a RAID is very much a specialized computer system: it has a processor, memory,and disks; however,instead of running applications, it runs specialized software designed to operate the RAID.

问: RAID解决了什么问题?

> 答: 磁盘的可靠性和性能问题. 使用多快磁盘并行读写, 使用冗余磁盘确保数据不丢失.

问: RAID是如何实现的 ?

> 答: 对RAID的IO请求都会转化了对几个磁盘的IO请求(封装, 转发)

问: RAID有哪些实现 ?

> 答:
>
> | 实现        | 容量(n为数量, N为容量) | 可靠性         | 速度(单盘速度s) |
> | ----------- | ---------------------- | -------------- | --------------- |
> | RAID0       | n x N                  | 无             | s * N           |
> | RAID1(镜像) | n * N / 2              | 可以损坏一块盘 | 和单个磁盘持平  |
> |             |                        |                |                 |

问: 什么是文件?

> 答: inode, 提供了顺序和随机读写的接口. 文件头提供了初数据之外的全部信息.
>
> ![1534163323569](C:\Users\ADMINI~1\AppData\Local\Temp\1534163323569.png)

问: 什么是文件夹?

> 答: [{inode: "fileName"}]



问: 如何保证磁盘中的数据在断电或者系统宕机之后还能保持一致?

> 答: 磁盘数据写入是一个事务, 包括多个操作, 写头信息, 写入数据等, 需要保证事务的原子性. 那么问题的实质是如何实现一个磁盘更新的事务.
>
> 实现一: 定时扫描磁盘, 如果发现有不一致的状态, 然后修复. 缺点是太慢了
>
> 实现二: 提前写日志, 如果系统宕机, 那么可以从日志中得到足够的信息恢复