---
title: JAVA笔记 -- 访问权限控制
sidebar:
  - toc
date: 2019-12-17 13:16:00
tags: [Java]
categories: [Java基础]
---

## 访问权限控制

没有权限控制的时候，由于所有的接口都是可以访问的。当一个类库部分代码，发现有更好的方法解决的时候，可能其他接口会发生改动。这会导致另一个地方的引用该类库的程序发生崩溃。为了解决这种问题，权限管理就显得尤为重要了。

在Java中提供了**访问权限控制修饰词**。以供类库开发人员向客户端程序员说明哪些功能是可以用的，那也有是不可以用的。

### **一、包：库单元**

> **包**内含有一组类，他们在**单一的名字空间**之下被组织到了一起

#### **类名冲突?不存在的**

在程序中，如果需要其他包的类，就需要`导入`。
```java
import java.util.*;
```

之所以要导入，就是因为要提供一个管理名字空间的机制。所有类之间的名字是相互隔离的。如果在机器上编写了相同名字的类，那个**为每个类创建唯一的命名空间**就会非常重要。

> 单一文件中的代码，并不是不位于包中，而是已存在于**未命名包**中。

#### **代码组织**

**类库**实际上是一组类文件。每个文件，都有一个`public`类，以及**任意数量**的**非public**类。

使用`package`语句，这个语句必须放在除注释以外的第一句程序代码：
```java
//这是一句注释
package accsess.mypackage;

public class MyClass {
    // 假装里面有内容
}
```

如果其他地方需要用这个类，那个就需要用引入`import access.mypackage.*;`或者使用全名`access.mypackage.MyClass`

#### **创建一个独一无二的包名**

那个男孩不想要一个独一无二的包名呢？

java包的命名用域名的方式来命名。众所周知，域名是不会重复的，是唯一的。这样的好处是，可以减少重复，而且方便别人的记忆。

环境变量`CLASSPATH`可以提供查找包的位置并且是**全局的**！
> CLASSPATH=.;D:\JAVA\JPackage

承接上文内容，`java.util.*`和`my.mypackage.*`均存在一个类，叫做`Vector`那么，
```java
import java.util.*;
import my.mypackage.*;

public class LibTest{
    public void main(){
        //! 下面的这一行代码会报错，因为你，我，编译器都不知道是那个包里面的Vector类
        //! Vector V = new Vector();
        //因此要求使用全名，如下方的实例相同
        java.util.Vector V1 = new java.util.Vector();
        my.mypackage.Vector V2 = new my.mypackage.Vector();
    }
}
```

#### **定制工具库**

通过包就可以定制自己的专属工具库了
```java
//这里就举例一个Print的工具库吧！
//这里就可以使用港方编辑的静态类来解决问题了
public class Print {  
    public static void print(Object obj) {
        System.out.println(obj);
    }
    public static void print() {
        System.out.println();
    }
    public static void printnb(Object obj) {
        System.out.print(obj);
    }
    public static PrintStream
    printf(String format, Object... args) {
        return System.out.printf(format, args);
    }
}
```
### **二、Java访问权限修饰词**

> 权限修饰词`public`, `private`和`protected`（包访问权限又是也被称为`friendly`）

#### **包访问权限**

包访问权限允许包内的各个类组合起来，以便使他们彼此之间可以轻易地相互作用。

类控制着哪些代码具有访问自己成员的权限。获取某成员的唯一途径是：

1. 使得该成员为public。无论是谁，无论在何地，都可以访问该成员。
2. 不加访问权限修饰词，将其他类放置在与该成员相同的包中。
3. 继承。
4. 提供**访问器**(**accessor**)和**变异器**(**mutator**)**方法**读取和改变数值。如`xxx.get()`, `xxx.set(Object obj)`。

#### **public：接口访问权限**

使用关键字`public`以后，意味着成员对**每个人都是可用**的。


> Java并不总是将当前目录视为查找行为的起点之一。如果CLASSPATH缺少`.`作为路径之一的话，Java就不会查找那里。

#### **默认包**

```java
//文件一：Cake.java
class Cake{
    public static void main(String[] args){
        Pie x = new Pie();
        x.f();
    }
}

//文件二：Pie.java
class Pie{
    void f(){
        System.out.println("Pie.f()");
    }
}

//文件一、二均位于同一文件夹下
```

尽管这两个文件看似没啥关系，但是，他们的确享有**包的访问权限**。Java将这样的文件看作是隶属于该目录的默认包中。该目录中的所有其他文件拥有包的访问权限。

#### **private：你无法访问**

除了包含这个成员的类除外，其他的任何类都无法访问该成员。

> 由于对`private`关键字没有啥感情，暂且写这么多吧！

#### **protected：继承访问权限**

基类的创建者希望有某个特定的成员，把对它的访问权限赋予派生类而不是所有类。那么就需要`protected`来完成这样的工作。

> 在后续的继承中会说明具体使用情况的QAQ

### **三、接口和实现**

访问权限的控制常称为是**具体实现的隐藏**。把数据和方法包装进类中，以及具体实现的隐藏。包装后的结果是，一个同时带有特征和行为的数据类型。
