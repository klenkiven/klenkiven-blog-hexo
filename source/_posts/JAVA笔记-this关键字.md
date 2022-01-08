---
title: JAVA笔记 -- this关键字
sidebar:
  - toc
date: 2019-12-07 10:46:00
tags: [Java]
categories: [Java基础知识]
---

## this关键字
### 一、 基本作用

> 在当前方法内部，获得**当前对象的引用**。在引用中，调用方法不必使用`this.method()`这样的形式来说明，因为编译器会自动的添加。

**必要情况**：

1. 为了将对象本身返回
    ```java
    public class Leaf{
        int i = 0;
        Leaf increment(){
            i++;
            return this;    //明确指出当前对象引用，返回当前对象
        }
    }
    ```
2. 引用外部工具传递方法时，为了将自身传递到外部方法
   ```java
    class Peeler{
        static Apple peel(Apple apple){
            //remove pell
            return apple;
        }
    }
    class Apple{
        Apple getPeeled(){
            return Peeler.peel(this); //这里的this是必要的，将自身传递给外部方法
        }
    }
   ```
### 二、 在构造器中调用构造器

> 一个类可能有很多个构造器(**重载构造器**)，如果在一个构造器中调用另一个构造器，避免重复代码，就可以调用其他构造器。这时，就需要`this`关键字。

1. 调用构造器的时候，必须放在**起始处**
   ```java
    class CallConstructor(){
        CallConstructor(int i){
            System.out.println(i);
        }
        CallConstructor(String str){
            this(6);     //一定要放在起始处
            System.out.println(str);
            //! this(6);  //放在这里，编译器会报错
        }
    }
   ```
2. 调用构造器的时候，**只**能调用**一次**
   ```java
    class CallConstructor(){
        CallConstructor(int i){
            System.out.println(i);
        }
        CallConstructor(double n){
            System.out.println(n);
        }
        CallConstructor(String str){
            this(6);       //一定要放在起始处
            //! this(1.0); //放在这里编译器会报错，不可以调用两次
            //其实说白了也是调用构造器的时候，一定要放在开头
            System.out.println(str);
        }
    }
   ```
3. 除了构造器之外，其他方法**禁止**调用构造器
   ```java
    class CallConstructor(){
        CallConstructor(int i){
            System.out.println(i);
        }
        CallConstructor(double n){
            System.out.println(n);
        }
        void CommMethod(){
            //! this(6);     //这是错误的！一定不可以这么写
            System.out.println("Common Method");
        }
    }
   ```
### 三、 static的含义
> `static`顾名思义，就是静态的意思。这个关键字还会在后续继续探究。
1. static方法
   
    `static方法`就是没有this的方法。`static方法`不能调用非静态方法，反过来是可以的。
2. static方法具有**全局函数**的语义

