---
title: JVM之内存分类
date: 2018-10-31 10:14:06
tags: jvm
---

# 栈

是一个线程私有的内存区域, 每一个方法调用就会产生一个`stack frame`,  用于储存方法调用的各种信息

1. 局部变量表
2. 操作数
3. 方法出口

如果一个线程中的`stack` 超过大小, 那么就会抛出`java.lang.StackOverflowError`

可以通过 `-Xss` 参数调整`stack`的大小.

下面的代码默认的大小只能调用6000次

```java
	@Test
	public void testStackOverflow() throws Exception {
		int stackCounter = 0;
		empty(0);


	}




	private void empty(int counter) {
		System.out.println("current stack counter is " + String.valueOf(counter));
		empty(counter + 1);
	}


```

# 堆

堆是JVM中存储对象的内存, 所有线程共享, 是垃圾回收主要进行的区域. 在栈中创建的对象都会存储在堆中, 栈中只存储引用.

当堆中的内存空间耗尽时, 会抛出`java.lang.OutOfMemoryError`

下面这段代码会导致OOM, 把堆大小调整的小一点可以快速的看到效果 `-Xmx10M -Xms10M`



```java
	@Test
	public void testOOM() throws Exception {

		LinkedList<Object> objects = new LinkedList<>();

		while (true) {

			objects.add(new Object());



		}

	}


```

# 方法区

在操作系统中, 这个区域被称为 `code` 也就是存储我们源代码的区域, 所有线程共享. 这个区域如果太小, 可能会导致以下问题:



1. 系统启动时抛出OOM, 因为系统启动时会加载很多类, 所以如果系统的类太多, 在启动时直接OOM
2. 在系统运行时, 如果生成的动态代理类太多, 也会导致OOM

