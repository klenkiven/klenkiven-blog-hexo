---
title: JAVA笔记---方法
sidebar:
  - toc
date: 2019-12-29 19:44:02
tags: [Java]
categories: [Java基础知识]
---

## 方法的基础

### **1. return 语句的一些高级应用**

```java
public class Method{
    public static void main(Sting[] args){
        System.out.println(Method_re);
    }
    public static void Method_01{
        for(int i = 0; i < 10; i++){
            if(i == 5)
                return;  //这里的 return; 可以终止函数的运行不能运行下面的打印语句
        }
        System.out.println("Hello World!");
    }
    public static void Method_01{
        for(int i = 0; i < 10; i++){
            if(i == 5)
                break;  //这里可以终止 for 循环，但是不能终止函数，下面的打印语句依然会执行
        }
        System.out.println("Hello World!");
    }
}
```

## 方法的内存分配

1. 方法**只定义，不调用**，是不会执行的，并且在JVM中也**不会**给该方法分配`运行所属`的内存空间。
2. 在JVM内存主要有三块内存划分：
    * **方法区内存**
    * **堆内存**
    * **栈内存**
3. 关于栈数据结构
    * 栈：stack，是一种数据结构
      * 一个栈 最上方的元素叫做**栈顶元素**，最下面的元素叫做**栈底元素**
      * **栈帧**永远指向栈顶元素
      * 栈顶元素处于**活跃**状态，其他元素处于**静止**状态
      * 术语：
        * 压栈/入栈/push
        * 弹栈/出栈/pop
      * 栈数据储存特点：**先进后出，后进先出**
    * 数据结构反映的是数据的储存形态
    * 数据结构是独立的学科，不属于任何编程语言的范畴
    * JavaSE的**集合**章节，使用了大量的数据结构
    * 提前精通：**数据结构 + 算法**
    * 常见的数据结构
      * 数组
      * 队列
      * 栈
      * 链表
      * 二叉树
      * 哈希表/散列表
      * ... ...

