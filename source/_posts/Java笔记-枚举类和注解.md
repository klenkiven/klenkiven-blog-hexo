---
title: Java笔记---枚举类和注解
sidebar:
  - toc
date: 2020-02-28 14:14:09
tags: [Java]
categories: [Java基础知识]
---

## 一、枚举类

### 自定义枚举类

#### 方式一：JDK5.0之前自定义枚举类

``` java
class Seasons {
    //1. 声明Seasons对象的属性
    private  final String SeasonName;
    private  final String SeasonDescrp;

    //2. 私有化类的构造器， 并给对象赋值初始化
    private Seasons(String SeasonName, String SeasonDescrp) {
        this.SeasonDescrp = SeasonDescrp;
        this.SeasonName   = SeasonName;
    }

    //3. 提供当前枚举类的多个对象 public static final "枚举类的名字"
    public static final Seasons SPRING = new Seasons("SPRING", "春天");
    public static final Seasons SUMMER = new Seasons("SUMMER", "夏天");
    public static final Seasons AUTUMN = new Seasons("AUTUMN", "秋天");
    public static final Seasons WINTER = new Seasons("WINTER", "冬天");

    //4. 获取枚举类对象的属性
    public String getSeasonName() {
        return SeasonName;
    }
    public String getSeasonDescrp() {
        return SeasonDescrp;
    }

    @Override
    public String toString() {
        return "Seasons{" +
                "SeasonName='" + SeasonName + '\'' +
                ", SeasonDescrp='" + SeasonDescrp + '\'' +
                '}';
    }
}
```

调用这个类：

```java
public class TestEnum {
    public static void main(String[] args) {
        Seasons spring = Seasons.SPRING;
		System.out.println(spring);
    }
}
```

> Seasons{ SeasonName='SPRING', SeasonDescrp='春天'}

以上为该自定义枚举类的输出结果

#### 方式二：JDK5.0之后引入关键字`enum`定义枚举类

```java
enum Season {
    //多个对象之间用逗号，最后一个对象用分号结束
    SPRING("SPRING", "春天"),
    SUMMER("SUMMER", "夏天"),
    AUTUMN("AUTUMN", "秋天"),
    WINTER("WINTER", "冬天");

    //下面的声明和方式一并无多大差别
    //1. 声明Seasons对象的属性
    private  final String SeasonName;
    private  final String SeasonDescrp;

    //2. 私有化类的构造器， 并给对象赋值初始化
    private Season(String SeasonName, String SeasonDescrp) {
        this.SeasonDescrp = SeasonDescrp;
        this.SeasonName   = SeasonName;
    }
}
```

那么，枚举类为何没有重写toString()方法呢？

```java
public class TestEnum {
    public static void main(String[] args) {
        Season spring = Season.SPRING;
		System.out.println(spring);
        System.out.println(Season.class.getSuperClass());
    }
}
```

> SPRING
> class java.lang.Enum

以上呢就是输出结果了！可见，这个类中的toString()方法已经再Enmu类中重写过了

### Enum类中的常用方法

1. values()：返回枚举类型的对象数组。该方法可以很便利的遍历所有枚举值。
2. valuesOf(String objName)：根据提供的objName，返回一个对象名是objName的枚举类型的对象
3. toString()：返回当前常量的名称

```java
public class TestEnum {
    public static void main(String[] args) {
        Season spring = Season.SPRING;
        
        //toString()方法
		System.out.println(spring);
        
        //values()方法
        Season[] values = Season.values();
        for (Season temp : values) {
            System.out.println(temp);
        }
        System.out.println(Season.class.getSuperClass());
        
        //valueOf(String objName)
        //如果找不到与objName同名的对象时候，会抛出一个异常IllegalArgumentException
        Season winter = Season.valueOf("WINTER");
        System.out.println(winter);
    }
}
```

### 使用enum关键字定义的枚举类实现接口

#### 情况一：实现接口。再enum类中实现接口

```java
interface Info {
    //显示对象的名字
    void show();
}
enum Season implements Info {
    //多个对象之间用逗号，最后一个对象用分号结束
    SPRING("SPRING", "春天"),
    SUMMER("SUMMER", "夏天"),
    AUTUMN("AUTUMN", "秋天"),
    WINTER("WINTER", "冬天");

    //1. 声明Seasons对象的属性
    private  final String SeasonName;
    private  final String SeasonDescrp;

    //2. 私有化类的构造器， 并给对象赋值初始化
    private Season(String SeasonName, String SeasonDescrp) {
        this.SeasonDescrp = SeasonDescrp;
        this.SeasonName   = SeasonName;
    }

    @Override
    public void show() {
        System.out.println("这是一个季节");
    }
}
```

#### 情况二：让枚举类的对象，分别实现接口中的方法

```java
interface Info {
    //显示对象的名字
    void show();
}
enum Season implements Info {
    //多个对象之间用逗号，最后一个对象用分号结束
    SPRING("SPRING", "春天"){
        @Override
        public void show() {
			//春天的方法
        }
    },
    SUMMER("SUMMER", "夏天") {
        @Override
        public void show() { }
    },
    AUTUMN("AUTUMN", "秋天") {
        @Override
        public void show() { }
    },
    WINTER("WINTER", "冬天") {
        @Override
        public void show() { }
    };

    //1. 声明Seasons对象的属性
    private  final String SeasonName;
    private  final String SeasonDescrp;

    //2. 私有化类的构造器， 并给对象赋值初始化
    private Season(String SeasonName, String SeasonDescrp) {
        this.SeasonDescrp = SeasonDescrp;
        this.SeasonName   = SeasonName;
    }
}
```

让每一个对象实现一次接口方法，使得每个对象有不同的动作。



## 二、注解

​	

Annotation是在代码里的**特殊标记**这些标记可以在编译时，类加载时被读取，并作出相应的处理。不改变原有逻辑的情况下，在程序中添加一些补充信息。

#### 应用

##### 应用一：在生成文档中使用注解（javadoc）

@author  @param @return @version 等等。

##### 应用二：编译时进行格式检查（JDK内置的三个注解）

@Override   表示限定重写父类方法，该注解只能用于方法

@Deprecated  表示所修饰的怨妇已经过时。通常是因为所修饰的结构危险或者存在更好的选择

@SuppressWarnings  抑制编译器警告

```java
class Person {
    public void walk() {
        System.out.println("人走路");
    }
	//下面的用于标记，这个方法已经过时了
    @Deprecated
    public void eat() {
        System.out.println("人吃饭");
    }
}
interface Info {
    void show();
}
class Student extends Person implements Info {
	//标记用于重写父类中的方法，以及实现接口中的方法
    @Override
    public void walk() { System.out.println("学生走路"); }
    @Override
    public void show() { System.out.println("我是学生"); }
    
    public static main(String[] args) {
        
        //在代码中，创建的变量未使用会有警告Waring
        //这里的unused就是表示忽略，没使用这个警告的
        @SuppressWarnings("unused")
        int num = 100;
        //System.out.println(num);
        
        //下面的这种"rawtypes"适用于使用泛型的时候并没有选取类型的警告
        @SuppressWarnings({"unused", "rawtypes"})
        ArrayList list = new ArrayList();
    }
}
```

##### 应用三：跟踪代码依赖性，实现代替配置文件功能

Servlet3.0提供了注解，使得不需要在web.xml文件中进行Servlet的部署

![image-20200226121316865](https://github.com/klenkiven/Blogs/raw/master/img/EnumAndAnnotation-Annotation-1.png)


#### 自定义注解

1. 自定义新注解使用 **@interface** 关键字
2. 定义 Annotation 成员变量时，以 ***无参数方法*** 的形式来声明。
3. 只有一个参数成员，建议使用**value**命名
4. 如果注解没有成员表明是一个**标识作用**
5. 使用注解的时候，**需要设定一个值**，如果已经有了默认值值，那么，就可以不用手写默认值了。
6. 自定义的注解，需要配上信息处理流程（使用**反射**）才有意义。
7. 通常会指明两个元注解，@Retention 和 @Target

```java
public @interface CustomAnnotation {
    //只有一个值的话值的名字叫做value就很好
    //String value();
    //当然也可以使用default设置为默认值
    String value() default "Hello";
}
```

#### JDK中的元注解

+ 元注解：对于现有注解进行修饰的注解。

+ JDK5.0 提供了4个标准的元注解(**meta-annotation**)类型：

  + **Retention**：指明所有修饰的 **Annotation** 的生命周期，其中包含了一个**RetentionPolicy**类型的成员变量。<详见JDK API>
    + 相关的Java SE API![image-20200226221922068](https://github.com/klenkiven/Blogs/raw/master/img/EnumAndAnnotation-Annotation-2.png)
    + 从上面可以看出只有 **RUNTIME** 才可以使用反射
    
  + **Target**：用于修饰 **Annotation** 的定义，用于 *指定*  被修饰的 **Annotation** 能用于修饰那些程序元素
    
    + 相关的Java SE API![image-20200226222538396](https://github.com/klenkiven/Blogs/raw/master/img/EnumAndAnnotation-Annotation-3.png)
    
  + Documented：用于指定被该元注解修饰的 **Annotation** 类将会被javadoc提取为相应的文档，默认情况下**Javadoc**是不包括注解的

  + Inherited：被它修饰的 **Annotation** 将具有***继承性***。如果某个类使用了被@Inherited标记的Annotation那么子类中也会继承父类中的，这个Annotation。

#### 通过反射获取注解信息

#### JDK8 中注解的新特性

可重复注解

```java
//JDK8.0之前想要使用重复注解的方法
public @interface MyAnnotations {
    MyAnnotation[] value();
}
public @interface MyAnnotation {
    String[] value();
}

```

```java
@MyAnnotations({
    @MyAnnotation(value = "hello"),
    @MyAnnotation(value = "hi")
})
class Person {
    public void walk() {
        System.out.println("人走路");
    }
	//下面的用于标记，这个方法已经过时了
    @Deprecated
    public void eat() {
        System.out.println("人吃饭");
    }
}
```

1. 在MyAnnotation上声明@Repeatable，成员值为MyAnnotations.class
2. MyAnnotation的Target和Retention与MyAnnotations相同 

```java
//这个是目标注解
import java.lang.annotation.*;
//使用注解@Rpeatable与这个注解的容器注解产生关联
@Repeatable(MyAnnotations.class)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    String value() default "hello";
}

//这个是目标注解的容器注解
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotations {
    MyAnnotation[] value();
}

//这是使用
@MyAnnotation("Hello")
@MyAnnotation("hi")
class Person {
    public void walk() {
        System.out.println("人走路");
    }
	//下面的用于标记，这个方法已经过时了
    @Deprecated
    public void eat() {
        System.out.println("人吃饭");
    }
}
```

  通过上下的比较，可以发现，JDK8的新特性是对原来注解**嵌套**的简化。

