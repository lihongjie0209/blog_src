---
title: Java之内存模型
date: 2018-08-10 21:48:31
tags:
    - Java
    - 内存管理
---

问: 什么是Java内存模型?

> 答: 由程序员自己控制一部分JVM代码优化的过程(只保证在单线程). JVM可以做到在指令级别内存读写优化并保证单线程情况下的语义不变.

原子性: new

可见性: 

顺序

---

问: 如何控制JVM代码优化?

> 答: 
>
> | 方法         | 反优化                        | 优点                                                   |
> | ------------ | ----------------------------- | ------------------------------------------------------ |
> | final        |                               |                                                        |
> | volatile     | 直接读写主存, 而不使用CPU缓存 | 只要volatile字段更新, 那么后续的线程读取确保是最新的值 |
> | synchronized |                               |                                                        |
>
> 

问: 非volatile字段与volatile字段有什么区别?

>答:
>
>
>
> ![非volatile字段, 每个CPU都有缓存](http://liimg.oss-cn-shenzhen.aliyuncs.com/18-8-11/41664555.jpg)
>![volatile字段直接读写主存](http://liimg.oss-cn-shenzhen.aliyuncs.com/18-8-11/14701928.jpg)



