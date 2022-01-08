---
title: Java笔记---成员初始化
sidebar:
  - toc
date: 2019-12-29 19:41:58
tags: [Java基础知识]
categories: [Java基础知识]
---

## **成员初始化**
Java尽力保证所有变量可以在使用前可以初始化。
```java
void f(){
    int i;
    System.out.println(i);
    //! i++;   //开幕雷击：这里就报错了，会告诉你变量 i 未初始化
}

//Output: 0
```
这说明，数据成员在创建之初是有初始值的。但这并不代表java为数据成员提供了默认值。

### **指定初始化**
1. 直接赋值法
   ```java
    int i = 0;
    char ch = 'a';
     ```
2. 调用方法对数据成员赋值
   ```java
    public class MethodInit1{
        int i = f();    //调用方法对 i 赋值
        int f() { return 11; }
    }

    public class MethodInit2{
        int i = f();
        int j = g(i);   // i 变量已经被初始化可以这样做
        int f(){ return 11; }
        int g(int n){ return n*5 }
    }

    public class MethodInit3{
        int j = g(i);   // i 变量还未被初始化会报错
        int i = f();
        int f(){ return 11; }
        int g(int n){ return n*5 }
    }
   ```
    可见上述的程序运行情况取决于成员变量的初始化顺序，如果上面调用下面的成员变量`向前引用`就会发出警告。
