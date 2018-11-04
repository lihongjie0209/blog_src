---
title: JVM之类文件
date: 2018-11-03 22:32:01
tags:JVM Class文件
---

# 前言

# 结构

## 无符号数

### 魔数

用于表示Java的类文件, 在类文件的前四个字节

```
cafe babe
```



### 版本号

魔数之后的后四个字节

```
0000 0034 表示的的是JDK1.8
```



### 常量池大小

版本号之后的两个字节

```
0036 表示有53个常量
```

### 常量池(大小依赖于源代码)

1. 源代码中的常量, 如字符串
2. 类和接口的全限定名
3. 字段名称和描述符
4. 方法名称和描述符

类, 字段, 方法都是由字符串变量组合而成的.

比如说我们有以下常量池, 反编译`HelloWorld.class` 得到的

```

  Constant pool:
   #1 = Methodref          #7.#24         // java/lang/Object."<init>":()V
   #2 = Fieldref           #25.#26        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #27            // hello world
   #4 = Methodref          #28.#29        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Fieldref           #6.#30         // cn/lihongjie/HelloWorld.i:I
   #6 = Class              #31            // cn/lihongjie/HelloWorld
   #7 = Class              #32            // java/lang/Object
   #8 = Utf8               i
   #9 = Utf8               I
  #10 = Utf8               <init>
  #11 = Utf8               ()V
  #12 = Utf8               Code
  #13 = Utf8               LineNumberTable
  #14 = Utf8               LocalVariableTable
  #15 = Utf8               this
  #16 = Utf8               Lcn/lihongjie/HelloWorld;
  #17 = Utf8               main
  #18 = Utf8               ([Ljava/lang/String;)V
  #19 = Utf8               args
  #20 = Utf8               [Ljava/lang/String;
  #21 = Utf8               <clinit>
  #22 = Utf8               SourceFile
  #23 = Utf8               HelloWorld.java
  #24 = NameAndType        #10:#11        // "<init>":()V
  #25 = Class              #33            // java/lang/System
  #26 = NameAndType        #34:#35        // out:Ljava/io/PrintStream;
  #27 = Utf8               hello world
  #28 = Class              #36            // java/io/PrintStream
  #29 = NameAndType        #37:#38        // println:(Ljava/lang/String;)V
  #30 = NameAndType        #8:#9          // i:I
  #31 = Utf8               cn/lihongjie/HelloWorld
  #32 = Utf8               java/lang/Object
  #33 = Utf8               java/lang/System
  #34 = Utf8               out
  #35 = Utf8               Ljava/io/PrintStream;
  #36 = Utf8               java/io/PrintStream
  #37 = Utf8               println
  #38 = Utf8               (Ljava/lang/String;)V
```



我们拿其中的`println`进行说明

```java
System.out.println("hello world");
```

![](http://liimg.oss-cn-shenzhen.aliyuncs.com/18-11-4/70851148.jpg)









### 类文件访问权限

1. 类文件的类型(ACC_${TYPE}  这个在源码反编译的时候一眼就可以看出来)
   1. 接口
   2. 抽象类
   3. 注解
   4. 枚举

2. 类文件的访问权限
   1. public
   2. final

### 继承关系

1. this_class 当前类
2. super_class 父类
3. interface 实现的接口



### 字段集合

1. 标志位(权限位)
   1. public
   2. static
   3. private
   4. .....
2. 字段信息
   1. 字段名
   2. 字段类型

### 方法集合

1. 标志位
2. 名称
3. 描述符



# 总结

类文件中的, 所有的信息都存储在常量池中, 而JVM指令只需要以常量池中的数据作为参数运行就可以了,

