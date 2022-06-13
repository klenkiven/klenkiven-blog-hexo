---
title: Java继承机制
sidebar:
  - toc
date: 2022-06-04 14:42:48
tags: [Java]
categories: [Java基础知识]
---

## 概述

本章将学习面向对象编程的另一个概念：继承（inheritance）。继承的基本思想是，可以基于已有的类创建新的类。
> **Note：** 一般而言，子类来继承超类，目的不是使用超类的方法，而是去修改超类的方法，或者是去扩展超类的方法。因此，继承一般很少去使用，如果处于使用为目的，那就可以使用依赖关系进行注入，或者使用代理模式来使用这些类。

## 类、子类、超类

“is-a” 关系是继承的一个明显特征。

> **C++:** Java 与 C++ 定义继承类的方式十分相似。Java 用关键字 extends 代替了 C++
中的冒号（：）。在Java 中，所有的继承都是公有继承，而没有C++ 中的私有继承和保
护继承。

### 定义子类

已存在的类称为**超类**(superclass)、 基类（ base class) 或超类（parent class)；新类称为**子类**（subclass)、 派生类(derived class) 或孩子类（child class)。

```java
public class A extends B {  }
```

子类能够继承超类所有的方法和属性，但是却不能访问超类的私有方法和属性。因此，private保证了子类和超类之间的封装性。protected能够使得子类和超类之间能够相互调用，保证了超类和子类与其他类之间的封装性。

> **Note:** 定义子类的时候，需要指出**不同之处**，因为上面说到，子类继承了超类的所有方法和属性，那么应该将一般的方法放在超类里面，而将更特殊的方法放在子类中，这种将通用的功能抽取到超类的做法在面向对象程序设计中十分普遍。

### 覆盖方法

子类如果声明和超类方法签名一致的方法，那么子类调用此方法就需要关键字`super`来保证调用超类的方法，而不是子类的。

> **Note:** 有些人认为 super 与 this 引用是类似的概念， 实际上，这样比较并不太恰当。这是因为 super 不是一个对象的引用，不能将 super 赋给另一个对象变量， 它只是一个指示编
译器调用超类方法的**特殊关键字**。

### 子类构造器

如果子类的构造器没有显式地调用超类的构造器，则将自动地调用超类默认（没有参数）的构造器。如果超类没有不带参数的构造器，并且在子类的构造器中又没有显式地调用超类的其他构造器，则 Java编译器报告错误。

> 回忆一下，关键字this有两个用途∶一是**引用隐式参数**，二是**调用该类其他的构造器**。同样，super关键字也有两个用途∶一是**调用超类的方法**，二是**调用超类的构造器**。在调用构造器的时候，这两个关键字的使用方式很相似。调用构造器的语句只能作为另一个构造器的第一条语句出现。构造参数既可以传递给本类（this）的其他构造器，也可以传递给超类（super）的构造器。

### 继承层次

由一个公共超类派生出来的所有类的集合被称为**继承层次（ inheritance hierarchy )**，在继承层次中， 从某个特定的类到其祖先的路径被称为该类的**继承链 ( inheritance chain)**。

{% image https://klenkiven-blog-image.oss-cn-zhangjiakou.aliyuncs.com/img/Java-Inheri-Hirearchy.png 继承层次 %}

> C++中一个类可以有多个超类。Java不支持多重继承，但是提供了一些类似多重继承的功能 -- 接口。看下一章6.1节有关接口的讨论。

### ✨多态

多态是面向对象程序设计语言一个很重要的特性。上文说到，继承最重要的一个特征就是"is-a"规则。这个规则另一个表述就是“**替换原则（substitution principle）**”。这个原则指出，程序中出现超类对象的任何地方都可以使用子类对象替换。

Java中对象变量是多态的。

```java 对象变量的多态性
public class B {}
public class A extends B {}
// 声明变量
A var1 = new A();
B var2 = var1;
// 数组的例子
B[] bArr = new B[10];
bArr[0] = new A();
```
> **⚠️Warning:** 在 Java 中，子类数组的引用可以转换成超类数组的引用， 而不需要采用强制类型转换。
> ```java ⚠️错误的例子
> Super[] s = new Supper[10];
> Sub[] sub = s;
> sub[0] = new Sub();
> // 下面的语句就会出现运行时错误
> s[0].getSuperAttribute();
> ```

### 理解方法的调用过程

虚拟机调用方法也是一个很有趣的话题，具体的执行流程如下：

1. 编译器查看对象的声明类型和方法名
2. 编译器确定方法调用中提供的参数类型
3. 如果是private、static、final方法或者构造器，那么编译器可以准确的知道应该调用那个方法。这成为**静态绑定（static binding）**
4. 程序运行并且采用动态绑定调用方法时，虚拟机必须调用与x所引用对象的实际类型对应的那个方法。

当然上面这样的动态绑定的方法增加了系统开销，虚拟机会自动为类计算了一个**方法表（method table）**。那么使用方法表解析过程为：

1. 虚拟机获取对象**实际类型**的方法表
2. 虚拟机查找定义方法签名的类
3. 虚拟机调用这个方法

### 阻止继承：final类和方法

将方法或者类声明为`final`的意义：确保它们不会再子类中改变语义。

**Note:** 在早期，为了避免动态绑定带来的开销，会使用final关键字。编译器会识别关键字，并且进行优化处理，这种处理叫做**内联（inline）**。

### 强制类型转换

进行强制类型转换的原因是：要在暂时忽略对象的实际类型之后使用对象的全部功能。

```java 强制类型转换
Super s = new Sub();
Sub s = (Sub) s;
```

{% noteblock 强制类型转换注意事项 %}
在进行强制类型转换之前，先查看是否能成功转换
```java 典型示范
if (staff[1] instanceof Manager) {
    boss = (Manager) staff[1];
    // Do something
}
```
{% endnoteblock %}

综上所述，
- 只能在继承层次内进行强制类型转换
- 再将超类强制转换成子类之前，需要使用`instanceof`进行检查

### 抽象类

如果自下而上在类的继承层次结构中上移，位于上层的类更具有通用性，甚至可能更加抽象。从某种角度看， 祖先类更加通用， 人们只将它作为派生其他类的基类，而不作为想使用的特定的实例类。

{% noteblock 提示 %}
建议尽量将通用的字段和方法（不管是否是抽象的）放在超类中（不管是不是抽象类）。
{% endnoteblock %}

**抽象类不能实例化。**需要注意的是，可以定义一个抽象类的*对象变量*，但是这样一个变量只能引用非抽象子类的对象。

### 受保护的访问

在类封装中会使用`private`关键字来标识，但是，如定义所见，private标识的类、方法、字段，只能由这个类自己来访问，包括派生类也就是子类也不能访问。那么就需要一种访问机制来为继承关系提供一种访问权限，那么这个访问就是受保护的访问，即`protected`。

在实践中，还需要谨慎使用受保护的字段。如果想要修改这个类的内容，那么就可能会导致其他派生类因此导致出现问题。受保护的方法更具有实践意义。需要限制某个方法的使用，但是子类还能使用，那么就需要将他声明为`protected`的方法。

## Object：所有子类的超类

Object是所有对象的超类，但是没有显式去声明`public class A extends Object`。由于Java中所有对象都是这个类派生的，所以这个类很重要。

Object类型的变量只能用于作为各种值的泛型容器。因为Java中存储的对象只是对象的一个引用，可以理解为一个指针。C++虽然没有所有类的根类，但是每个指针都可以转换成`void*`指针。

这个类大部分API都是用于并发编程使用，例如，`wait()`, `notify()`, `yield()`等等。但是还有很多API与平时的开发编程息息相关，包括，`equals()`, `toString()`, `hashCode()`方法。

### `equals()`方法

Object类中的equals方法用于检验一个对象是不是等于另一个对象。因此**默认行为**就是：如果两个对象引用相等，那么这两个对象就肯定相等。

但是，大多数业务上检测对象的相等性都是基于状态检测对象的相等性，如果对象状态相等，那么对象就是相等的。因此，再定义子类的时候通常会重写`equals()`。

{% noteblock ⚠️注意 %}
为了防止对象内的成员变量为null，需要使用Objects工具类的`Objects.equals()`方法。
```java 示范代码
// 如果有一方为null，返回false
return Objects.equals(this.name, other.name);
```
{% endnoteblock %}

### 相等性测试与继承

如果隐式和显式的参数不属于同一个类，equals方法该怎么处理？

1. 使用`instanceof`关键字来判断是不是属于同一个类
2. 使用`getClass()`方法来判断是不是属于同一个类

因为Java语言规范要求equals方法具有下面的性质：
- 自反性
- 对称性
- 传递性
- 一致性
- 对于任意非空的引用`x`，`x.equals(null)`应该返回`false`

按照这个性质两种方案各有各自的缺陷：

第一种方案，违背了**对称性**的性质。例如，Super类是Sub的超类，那么
```java 错误示范
Super s = new Super();
Sub sub = new Sub();
// 下面的案例不会出现问题，因为sub确实 'is-a' Super
s.equals(sub);
// 但是下面的就会出现问题
sub.equals(s);
```
因为子类会复写equals方法，而且使用了`instanceof`关键字，所以会出现问题。

第二种方案，违背了**替换原则**。如果判断TreeSet和HashSet判断是不是同一个Set，这样的场景使用`getClass()`方法就会出现问题。

综上所述，

1. 如果**子类能够拥有自己的相等概念**， 则对称性需求将强制采用 getClass 进行检测
2. 如果**由超类决定相等的概念**，那么就可以使用 imtanceof进行检测， 这样可以在不同
子类的对象之间进行相等的比较

### 编写`equals()`方法论

1. 显式参数命名为 otherObject, 稍后需要将它转换成另一个叫做 other 的变量。
2. 检测 this 与 otherObject 是否引用同一个对象
3. 检测 otherObject是否为 null，如果为 null，返回 false。
4. 比较 this 与 otherObject 是否属于同一个类。
5. 将 otherObject 转换为相应的类类型变量
6. 现在开始对所有需要比较的域进行比较了。

```java 示范代码
@Override
public boolean equals(Object otherObject) {
    if (this == otherObject) return true;
    if (otherObject == null) return false;
    // 判断是不是同一个类
    // 1. if (getClass() != otherObject.getClass()) return false;
    // 2. if (!(otherObject instanceof ClassName)) return false;
    ClassName other = (ClassName) otherObject;

    return super.equals(other) 
        && // 判断需要比较的域
        && ... ;
}
```

对于数组类型的域， 可以使用静态的`Arrays.equals`方法检测相应的数组元素是否相等。

### `hashCode()`方法

**散列码（hash code）**是由对象导出的一个整型值。

```java 字符串计算Hash
int hash = 0;
for (int i = 0; i < length0；i++)
    hash = 31 * hash + charAt(i);
```

如果重新定义`equals`方法，就必须重新定义`hashCode`方法，以便用户可以将对象插入到散列表。

计算普通对象的Hash，可以使用下面的代码来计算：

```java 计算Hash
public int hashCode() {
    return Objects.hash(name, salary, hireDate);
}
```

> **Note**：equals 与 hashCode 的定义必须一致∶如果x.equals(y)返回 true，那么x.hashCode()就必须与 y.hashCode()具有相同的值。

{% noteblock 源码分析 %}
Objects.hash方法，本质上使用了一个可变长参数列表，然后使用Arrays.hashCode方法来实现的。
{% endnoteblock %}

### `toString` 方法

这个方法很常见，主要用于直接观察对象的内部状态，这里不多赘述。

## 枚举类

```java 
public enum Size { SMALL , MEDIUM, LARGE, EXTRAJARGE };
```
实际上， 这个声明定义的类型是一个类， 它刚好有 4 个实例， 在此尽量不要构造新对象。

如果需要的话， 可以在枚举类型中添加一些构造器、 方法和域。当然，构造器只是在构
造枚举常量的时候被调用。

```java
public enum Size {
    SWALL("S"),MEDIUN("W"),LARCE("1"),EXTRA_LARGE("XL");
    private String abbreviation;
    private Size(String abbreviation) { this.abbreviation = abbreviation;}
    public String getAbbreviation(){ return abbreviation;}
}
```

所有的枚举类型都是 Enum 类的子类。

```java
Size s = Enum.valueOf(Size.class,"SWlL");
```

每个枚举类型都有一个静态的 values方法，它将返回一个包含全部枚举值的数组。

```java
Size[] values = Size.values()
```

ordinal方法返回enum声明中枚举常量的位置，位置从0开始计数。