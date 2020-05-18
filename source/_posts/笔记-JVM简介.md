---
title: 笔记-JVM简介
date: 2020-05-18 11:12:10
categories: 后端学习
tags:
- [JVM]
- [JVM笔记]
---

JVM简介，本笔记来自宋红康JVM课件，掘金上别人的[笔记](https://juejin.im/post/5e71c5c96fb9a07c98550df2)

<!-- more -->

# JVM简介

## JVM所处的位置

![JVM位置](笔记-JVM简介/JVM的位置.png)

## 市面上的一些JVM

- SUN Classic
- Exact VM
- HotSpot VM ：HotSpot指热点代码探测技术
- BEA JRockit：(BEA 已被Oracle收购) 专注于服务端应用，世界最快的jvm之一
- IBM J9
- Taobao JVM: 目前已经在淘宝、天猫上线，替换了Oracle官方JVM；
- Graal VM: Oracle 2018年4月公开，口号 Run Programs Faster Anywhere.最可能替代HotSpot的产品

### Android虚拟机 DVM

- 谷歌开发，基于Android，在2.2中提供了JIT
- 只能称作虚拟机 不能称为java虚拟机，他没有遵循Java虚拟机规范
- 基于寄存器架构，效率高，但是跟硬件耦合度比较高
- 不能直接执行class文件，执行的是dex文件
- 5.0使用支持提前编译的ART VM替换Dalvik VM

### JVM体系概览

![JVM体系概览](笔记-JVM简介/JVM体系概览.png)

### java代码执行流程

java程序--（编译）-->字节码文件--（解释执行）-->操作系统（Win，Linux，Mac JVM）

### 两种指令集架构

由于跨平台的设计，**java的指令都是根据栈来设计的**，不同平台CPU架构不同，所以不能设计为基于寄存器的架构。
**栈的指令集架构**：跨平台性、**指令集小、指令多；**执行性比寄存器差
**寄存器的指令集架构**：指令少

```shell
//查看指令集命令代码
cd out/production/类根目录
javap -v StackStruTest.class

//打印程序执行的进程
jps
```

### jvm生命周期

#### 1.启动

通过**引导类加载器**（bootstrap class loader）创建一个初始类（initial class）来完成的，这个类是由虚拟机的具体实现指定的。

#### 2.执行

- 一个运行中的java虚拟机有着一个清晰的任务：执行Java程序；
- 程序开始执行的时候它才运行，程序结束时它就停止；
- 执行一个所谓的Java程序的时候，真真正正在执行的是一个叫做Java虚拟机的进程。

#### 3.退出

- 程序正常执行结束
- 程序异常或错误而异常终止
- 操作系统错误导致终止
- 某线程调用Runtime类或System类的exit方法，或Runtime类的halt方法，并且java安全管理器也允许这次exit或halt操作
- 除此之外，JNI规范描述了用JNI Invocation API来加载或卸载Java虚拟机时，Java虚拟机的退出情况