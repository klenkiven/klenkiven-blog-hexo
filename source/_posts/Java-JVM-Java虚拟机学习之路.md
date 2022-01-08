---
title: '[Java][JVM] Java虚拟机学习之路'
sidebar:
  - toc
date: 2021-12-31 15:38:00
tags: [Java, JVM]
categories: [Java虚拟机]
cover: https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/JVM-内存结构.jpg
---

## 什么是Java虚拟机？

Java为了摆脱平台的束缚，为此Java语言运行于Java虚拟机上，实现了“一次编译，处处运行”的理想。同时Java虚拟机提供了良好的内存管理和访问机制，也就是使用了垃圾回收机制，同时严格限制了指针的使用（引用类型），因此避免了许多内存泄漏的问题。

## 为什么要使用Java虚拟机？

众所周知，C/C++语言可以编译为可执行文件，可执行文件可以在某个操作系统上运行，但是，如果跨操作系统，或者跨CPU架构就会出现很多问题，很难达到跨平台性。但是Java却提出了“一次编译处处运行”的口号，为什么能够达到这样的效果，其原因就是使用了Java虚拟机。在操作系统上再增加一层抽象，达到了一处编译（编译为字节码），处处运行（运行在不同OS不同CPU架构上的虚拟机上）。

既然如此，就会引出另一个问题，**性能**。那么Java性能问题怎么样呢？很显然因为增加了一层软件，Java虚拟机，Java语言的运行效率肯定难以高于C/C++的运行效率。

但是难道就没有解决办法了吗？答案是否定的，今天Java虚拟机使用了许多的技术来提升Java语言的运行效率，例如，JIT、AOT技术，就可以在运行时，或者是运行前对Java代码进行编译（后端编译器）从而达到原生应用的执行速度。这当然都是后话了。

## 如何学习Java虚拟机

Java虚拟机包括众多的主题，那么如何学习Java虚拟机也是一个问题，参考《深入理解Java虚拟机》整理了Java虚拟机学习的重要主题，同时我也会将每周的学习笔记公布到博客中。

Java虚拟机主要内容：

* 内存的自动分配
  > 如何去理解Java虚拟机的内存分配策略，内存区域的分配含义？
  >
  > 如何去理解垃圾回收机制？
  >
  > 垃圾回收算法有哪些？常见的垃圾回收器有哪些，如何去选择这些垃圾回收器？
  >
  > 性能监控工具和故障检测工具有哪些？如何去使用这些工具解决一些实际案例？
  >
* 虚拟机执行子系统
  > Class文件的结构是怎么样的？具体的字段有什么含义？
  >
  > JVM的类加载机制是怎么样的？什么是双亲委派机制？
  >
  > 为什么Tomcat破坏了双亲委派机制？怎么破坏的？
  >
  > 字节码的执行引擎由哪些部分组成？
  >
  > Java为什么说是一种动态类型语言？在哪里体现了动态性？
  >
* 高效并发
  > 这个部分其实更多的在Java并发编程中有所体现，因为Java虚拟机的内存模型，线程的内存模型很大程度上影响了并发编程的理论，因此这里不会过多的停留，可以参考《[Java并发编程的艺术](https://book.douban.com/subject/26591326/)》、《[Java并发编程实践](https://book.douban.com/subject/10484692/)》
  >

### 自动内存分配

* Java虚拟机内存区域
* 垃圾收集算法
* 垃圾收集器
* 性能调优
  * 性能监控，故障检测工具
  * 常见案例

### 虚拟机执行子系统

* 类文件结构（Class文件）
* JVM类加载机制
* 执行引擎

### 高效并发

* Java虚拟机内存模型
* 线程安全与锁优化

## 参考

* [深入理解Java虚拟机（第3版）](https://book.douban.com/subject/34907497/) 周志明
* [The Java Language Specification, Java SE 17 Edition](https://docs.oracle.com/javase/specs/jls/se17/html/index.html)
* [The Java Virtual Machine Specification, Java SE 17 Edition](https://docs.oracle.com/javase/specs/jvms/se17/html/index.html)