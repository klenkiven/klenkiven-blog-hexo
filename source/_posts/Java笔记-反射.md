---
title: Java笔记--反射
sidebar:
  - toc
date: 2020-03-13 13:46:00
tags: [Java]
categories: [Java基础]
---

## Java笔记--反射

### Java反射机制概述

+ Reflection (反射)是被视为**动态语言**的关键，反射机制允许程序在执行期借助于Reflection API取得任何类的内部信息，并能直接操作任意对象的内部属性及方法。
+ 加载完类之后，在堆内存的方法区中就产生了一个Class类型的对象(一个类只有一个Class对象)，这个对象就包含了完整的类的结构信息。我们可以通过这个对象看到类的结构。**这个对象就像一面镜子，透过这个镜子看到类的结构，所以，我们形象的称之为:反射。**

+ 正常方式和反射方式的对比：

  ![Reflection-1](https://github.com/klenkiven/Blogs/raw/master/img/Reflection-1.png)

#### Java反射机质研究及其应用

+ 在运行时判断任意一个对象所属的类
+ 在运行时构造任意一个类的对象
+ 在运行时判断任意一个类所具有的成员变量和方法
+ 在运行时获取泛型信息
+ 在运行时调用任意一个对象的成员变量和方法
+ 在运行时处理注解
+ 生成动态代理

#### 反射相关的主要API

+ **java.lang.Class**: 代表一个类
+ java.lang.reflect.Method：代表类的方法
+ java.lang.reflect.Field：代表类的成员变量
+ java.lang.reflect.Constructor：代表类的构造器

#### 反射初体验

```java
@Test
public void test1() {
    //1. 创建一个Person类
    Person p1 = new Person("ABC", 21);

    //2. 通过对象，调用其内部的方法，属性
    p1.age = 18;
    System.out.println(p1.toString());
    p1.show();

    //在Person类外部，不可以通过Person类内部的私有结构
    //name，Person(String name)
}

//反射之后，对于Person的操作
@Test
public void test2() throws Exception {
    //1. 通过反射创建Person类的对象
    //一个类的一个属性叫做.class这个属性就是Class这个类的一个实例
    Class clazz = Person.class;
    //使用反射找到这样的一个构造器
    Constructor cons = clazz.getConstructor(String.class, int.class);
    //使用构造器获得新的实例
    Object obj = cons.newInstance("Tom", 12);
    System.out.println(obj.toString());  //这里显示一个一个多态的特性
    Person p = (Person)obj;
    System.out.println(p.toString());

    //2. 通过反射，调用指定的属性和指定的方法
    //调用属性
    Field age = clazz.getDeclaredField("age");
    age.set(p, 10);
    System.out.println(p.toString());
    //调用方法
    Method show = clazz.getDeclaredMethod("show");
    show.invoke(p);

    System.out.println("*********************************");

    //通过反射可以调用Person类的私有结构。比如：name，Person(String name)
    //调用私有构造器
    Constructor pS = clazz.getDeclaredConstructor(String.class);
    pS.setAccessible(true);
    Object obj1 = pS.newInstance("Toms");
    Person p1 = (Person)obj1;
    System.out.println(p1.toString());

    //调用私有属性
    Field name = clazz.getDeclaredField("name");
    name.setAccessible(true);
    name.set(p1, "Thompson");
    System.out.println(p1.toString());

    //调用私有的方法
    Method showNation = clazz.getDeclaredMethod("showNation", String.class);
    showNation.setAccessible(true);
    String nation = (String)showNation.invoke(p, "中国");
    System.out.println(nation);
}
```

#### 反射机制和封装性辨析

Q1：通过直接new的方式或反射的方式都可以调用公共的结构，开发中用哪个？

A1：直接使用new的方式

Q2：什么时候会用到反射机制？

A2：不确定创建一个什么样的对象时，需要时用反射机制。

Q3：反射机制一面向对象中的封装性是不是矛盾的？如何看待两个技术？

A3：不矛盾。

### Class类的理解和获取Class实例

#### Class类的理解

1. 类的加载过程

   程序经过javac.exe命令以后，会生成一个或多个字节码文件(.class结尾)，此过程是编译过程。接着我们使用java.exe对某个字节码文件，进行解释运行。相当于讲某个字节码文件加载到内存中，此过程叫做**类的加载**。加载到内存中的类，叫做**运行时类**，就作为Class的一个实例。

2. 换句话说，Class的实例就对应着一个运行时类。

3. 加载到内存中的运行时类，会缓存一段时间。在此时间之内，我们可以通过不同的方式来获取此运行时类。

```java
//获取Class的实例方式
//前三种方式需要掌握
@Test
public void test3() throws ClassNotFoundException {
    //方式一：
    Class<Person> clazz1 = Person.class;  //可以避免强转之类的事情
    System.out.println(clazz1.toString());
    //方式二：通过运行时类的对象
    Person p1 = new Person();
    Class<? extends Person> clazz2 = p1.getClass();
    System.out.println(clazz2.toString());
    //方式三：调用Class的静态方法：forName(String classPath)
    Class<?> clazz3 = Class.forName("xyz.klenkiven.reflection.Person");

    System.out.println("clazz1 == clazz2\t" + (clazz1 == clazz2));
    System.out.println("clazz1 == clazz3\t" + (clazz1 == clazz3));

    //方式四：使用类的加载器（了解）
    ClassLoader classLoader = ReflectionTest.class.getClassLoader();
    Class<?> clazz4 = classLoader.loadClass("xyz.klenkiven.reflection.Person");
    System.out.println(clazz4.toString());

}
```

#### 那些类型可以有Class对象

1. class：

   内部类，成员（成员内部类，静态内部类），局部内部类，匿名内部类

2. interface：接口

3. []：数组

4. enmu：枚举

5. annotation：注解@interface

6. **基本数据类型**

7. **void**

```java
@Test
public void test4() {
    Class<?> c1 = Object.class;
    Class<?> c2 = Comparable.class;
    Class<?> c3 = String[].class;
    Class<?> c4 = int[][].class;
    Class<?> c5 = ElementType.class;
    Class<?> c6 = Override.class;
    Class<?> c7 = int.class;
    Class<?> c8 = void.class;
    Class<?> c9 = Class.class;

    int[] a = new int[10];
    int[] b = new int[100];
    Class<?> c10 = a.getClass();
    Class<?> c11 = b.getClass();
    //只要是数组的元素类型于维度一样，就是同一个Class
    System.out.println(c10 == c11);
}
```

#### 了解类的加载器

```java
@Test
public void test1() {
    //对于自定义类，使用系统类加载器加载器进行加载
    ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
    System.out.println(classLoader);
    //调用系统类加载的getParent()：获取扩展类加载器
    ClassLoader parent = classLoader.getParent();
    System.out.println(parent);
    //调用扩展类加载的getParent()：无法获取引导类加载器
    //引导类加载器主要负责加载java的核心类库，无法加载自定义类的
    ClassLoader parent1 = parent.getParent();
    System.out.println(parent1);

    ClassLoader classLoader1 = String.class.getClassLoader();
    System.out.println(classLoader);
}
```

#### 使用ClassLoader加载配置文件

```java
@Test
public void test2() throws Exception {

    Properties pros = new Properties();

    //读取配置文件方式一：
    //此时的文件默认在当前的Module下。
    //FileInputStream fis = new FileInputStream("src/jdbc.properties");
    //pros.load(fis);

    //读取配置文件方式二：
    //配置文件默认识别在当前Module下面的src下
    ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
    InputStream resourceAsStream = classLoader.getResourceAsStream("jdbc.properties");
    pros.load(resourceAsStream);


    String user = pros.getProperty("user");
    String password = pros.getProperty("password");

    System.out.println("user = " + user + "  password = " + password);

}
```

### 创建运行时类的对象

##### 使用 `newInstance()` 创建要求：

1. 使用这个方法一定需要有一个空参构造器
2. 使用这个方法一定需要一个合适的访问权限，通常权限为`public`

##### 在javabean中要求提供一个public的空参构造器，原因：

1. 便于通过反射，创建运行时类的对象
2. 便于子类继承此运行时类时，默认调用 `super()` 时，保证父类由此构造器

```java
@Test
public void test02() {
    int num = new Random().nextInt(3);
    String classPath = "";
    switch (num) {
        case 0:
            classPath = "java.util.Date";
            break;
        case 1:
            classPath = "java.util.EnumSet";
            break;
        case 2:
            classPath = "xyz.klenkiven.reflection.Person";
            break;
    }
    try {
        Object obj = getInstance(classPath);
        System.out.println(obj.toString());
    } catch (Exception e) {
        e.printStackTrace();
    }
}

/*
创建一个指定类的对象；
classPath：指定类的全类名
 */
public Object getInstance(String classPath) throws Exception {
    Class<?> clazz = Class.forName(classPath);
    return clazz.newInstance();
}
```

### 获取运行时类的完整结构

 #### 获取当前运行时类的属性结构

+ 获取**属性结构**

```java
@Test
public void test1() {

    Class<Animal> clazz = Animal.class;
    
    //getFields():  只能获取运行时类及其父类的public访问权限的属性
    System.out.println("getDeclaredFields()");
    Field[] fields = clazz.getFields();
    for (Field f : fields) {
        System.out.println(f);
    }
    System.out.println();
    //getDeclaredFields():  能获取运行时类的所有的属性
    System.out.println("getDeclaredFields()");
    Field[] declaredField = clazz.getDeclaredFields();
    for (Field f: declaredField) {
        System.out.println(f);
    }
}
```

+ 权限修饰符  数据类型  变量名=变量值

```java
@Test
public void test2() {
    Class<Animal> clazz = Animal.class;
    Field[] declaredFields = clazz.getDeclaredFields();
    for (Field f: declaredFields) {

        //1. 权限修饰符
        int modifiers = f.getModifiers();
        System.out.print(Modifier.toString(modifiers) + "\t");

        //2. 数据类型
        Class<?> type = f.getType();
        System.out.print(f + "\t");

        //3. 变量名
        String name = f.getName();
        System.out.print(name);

        System.out.println();
    }
}
```

#### 获取运行时类的方法结构

+ 获取运行时类的**方法**

```java
@Test
public void test1() {
    Class<?> clazz = Animal.class;

    //获取运行时类的方法
    //getMethods():  获取当前运行时类及其父类所有的public访问权限的方法
    Method[] methods = clazz.getMethods();
    for (Method m: methods) {
        System.out.println(m);
    }
    System.out.println();

    //getDeclaredMethods():  获取当前运行时类所有的声明过的所有方法
    Method[] declaredMethods = clazz.getDeclaredMethods();
    for (Method m: declaredMethods) {
        System.out.println(m);
    }
}
```

+ @Xxxx 权限修饰符 返回值类型 方法名 (数据类型 参数名，...) throws XxxException

```java
@Test
public void test2() {
    Class<?> clazz = Animal.class;
    Method[] declaredMethods = clazz.getDeclaredMethods();
    for (Method m: declaredMethods) {
        //1. 获取方法声明的注解
        Annotation[] annotations = m.getAnnotations();
        for (Annotation a: annotations) {
            System.out.println(a);
        }

        //2. 获取权限修饰符
        System.out.print(Modifier.toString(m.getModifiers()) + "\t");

        //3. 获取方法返回值类型
        System.out.print(m.getReturnType().getName() + "\t");

        //4. 获取方法名
        System.out.print(m.getName());

        System.out.print("(");
        //5. 形参列表
        Class<?>[] parameterTypes = m.getParameterTypes();
        if (!(parameterTypes == null || parameterTypes.length == 0)){
            for (int i = 0; i < parameterTypes.length; i++) {
                if (i != 0)
                    System.out.println(", ");
                System.out.print(parameterTypes[i].getName() + " args_" + i);
            }
        }
        System.out.print(")\t");

        //6. 获取异常
        Class<?>[] exceptionTypes = m.getExceptionTypes();
        if (!(exceptionTypes == null || exceptionTypes.length == 0)){
            System.out.print("throws\t");
            for (int i = 0; i < exceptionTypes.length; i++) {
                if (i != 0)
                    System.out.print(" ,");
                System.out.print(exceptionTypes[i].getName());
            }
        }

        System.out.println();
    }
}
```

#### 获取运行时类的构造器结构
+ 获取运行时类的**构造器**
```java
@Test
public void test1() {

    Class<?> clazz = Animal.class;
    //getConstructors():  获取运行时类的public访问权限的构造器
    Constructor<?>[] constructors = clazz.getConstructors();
    for (Constructor c: constructors){
        System.out.println(c);
    }

    System.out.println();

    //getDeclaredConstructors():  获取当前运行时类声明过的所有的构造器
    Constructor<?>[] declaredConstructors = clazz.getDeclaredConstructors();
    for (Constructor c: declaredConstructors) {
        System.out.println(c);
    }
}
```

+ 其他的属性的获取，可以参考**方法部分**的获取方法

#### 获取运行时类的带有泛型的父类

+ 获取运行时类的父类

```java
@Test
public void test2() {
    Class<?> clazz = Animal.class;

    Class<?> superclass = clazz.getSuperclass();
    System.out.println(superclass);
}
```

+ 获取运行时类**带有泛型的父类**

```java
@Test
public void test3() {
    Class<?> clazz = Animal.class;

    Type genericSuperclass = clazz.getGenericSuperclass();
    System.out.println(genericSuperclass.getTypeName());
}
```

+ 获取运行时类带有泛型的**父类的泛型**

```java
@Test
public void test4() {
    Class<?> clazz = Animal.class;

    Type genericSuperclass = clazz.getGenericSuperclass();
    ParameterizedType parameterizedType = (ParameterizedType) genericSuperclass;
    //获取泛型类型
    Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
    //System.out.println(actualTypeArguments[0].getTypeName());
    //和下面的方法是等价的
    System.out.println(((Class)actualTypeArguments[0]).getName());
}
```

#### 获取运行时类实现的接口，所在包，注解等等

+ 获取运行时类**实现的接口**

```java
@Test
public void test5() {
    Class<?> clazz = Animal.class;

    Class<?>[] interfaces = clazz.getInterfaces();
    for (Class<?> c: interfaces) {
        System.out.println(c);
    }

    System.out.println();

    Class<?> superclass = clazz.getSuperclass();
    //获取运行时类父类实现的接口
    Class<?>[] interfaces1 = superclass.getInterfaces();
    for (Class<?> c: interfaces1) {
        System.out.println(c);
    }
}
```

+ 获取运行时类当前所在的**包**

```java
@Test
public void test6() {
    Class<?> clazz = Animal.class;

    Package aPackage = clazz.getPackage();
    System.out.println(aPackage);
}
```

+ 获取运行时类生命的**注解**

```java
@Test
public void test7() {
    Class<?> clazz = Animal.class;

    Annotation[] annotations = clazz.getAnnotations();
    for (Annotation a: annotations) {
        System.out.println(a);
    }
}
```

### 调用运行时类的完整结构

#### 调用运行时类中指定的属性

1. 获取Class类
2. 获取指定的属性
3. 修改指定的属性

```java
@Test
public void testField() throws Exception {
    Class<?> clazz = Animal.class;

    //创建运行时类的对象
    Animal a = (Animal)clazz.newInstance();

    //getField(), getDeclaredField():  获取指定的属性

    //getField(): 只能获取public访问权限的属性
    Field id = clazz.getField("id");
    //设置当前属性的值
    //set():  参数1：指明设置那个对象的属性值  参数2：将此对象设置为多少
    id.set(a, 1001);
    //获取当前属性的值
    //get():  参数：获取的是那个对象的属性值
    int aid = (int)id.get(a);
    System.out.println(aid);
}
```

> 上面的这种方法已经是过时或者是用处比较小的方法，一般的话使用下面的`getDeclaredField()`方法

```java
@Test
public void testField1() throws Exception {
    Class<Animal> clazz = Animal.class;

    Animal a = clazz.newInstance();

    //getDeclaredField():  获取运行时类的已经声明过的属性
    //PS. 这里的name是Animal类中的私有属性
    Field declaredField = clazz.getDeclaredField("name");

    //setAccessible(true):  保证当前属性是可访问的
    declaredField.setAccessible(true); //只要加上这一句，内裤都给你扒出来
    //只有这一句不行，会抛出一个异常 IllegalAccessException 得结合上一句使用
    declaredField.set(a, "Dog");

    //获取对象中的值
    String name = (String)declaredField.get(a);
    System.out.println(name);

}
```

#### 调用运行时类中指定的方法

1. 获取指定的方法
2. 保证方法一定是可访问的 `setAccessable(true)`
3. 调用方法的 `invoke()`
4. `invoke()` 方法会返回一个`Object的返回值`，但是可以强转为原方法对应的返回值类型

```java
@Test
public void testMethod() throws Exception {
    Class<Animal> clazz = Animal.class;

    Animal animal = clazz.newInstance();

    //获取指定的方法
    //参数1：指明获取的方法的方法名
    //参数2：指明获取方法的形参列表
    Method show = clazz.getDeclaredMethod("show", String.class);
    //保证方法一定是可访问的
    show.setAccessible(true);
    /*
     * 调用方法的invoke():
     * 参数1：殖民方法的调用者，一个实例
     * 参数2，3，4...：方法的形参列表
     */
    show.invoke(animal, "犬类");
    //invoke():  方法会返回一个Object的返回值，但是可以强转为原方法对应的类型
    String breed = (String)show.invoke(animal, "犬类");
    System.out.println(breed);

    //调用私有的静态方法
    Method showDesc = clazz.getDeclaredMethod("showDesc");
    showDesc.setAccessible(true);
    //如果调用的运行时类的方法没有返回值，此invoke()返回null
    Object returnVal1 = showDesc.invoke(animal);
    //由于是静态方法，在加载类的时候就已经加载到方法区里面了，因此这里就知道是Animal的静态方法
    Object returnVal2 = showDesc.invoke(null);
    System.out.println(returnVal1);
    System.out.println(returnVal2);
}
```

#### 调用运行时类中的指定构造器

```java
@Test
public void testConstructor() throws Exception {
    Class<Animal> clazz = Animal.class;
    //private Animal(String)
    //1. 获取构造器
    //参数：指明构造器的参数列表
    Constructor<Animal> declaredConstructor = clazz.getDeclaredConstructor(String.class);
    //2. 保证构造器是可访问的
    declaredConstructor.setAccessible(true);
    //3. 调用此构造器
    Animal dog = declaredConstructor.newInstance("Dog");

    System.out.println(dog);
}
```

### 附录：测试中使用到的类

#### Person.java

```java
public class Person {

    private String name;
    public int age;


    //Constructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    private Person(String name) {
        this.name = name;
    }
    public Person() {
        System.out.println("Person()");
    }


    //Special Method
    public void show() {
        System.out.println("我是一个人");
    }
    private String showNation(String nation) {
        System.out.println("我是一个" + nation + "人");
        return nation;
    }


    //Setters, Getters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }


    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

#### Animal一系列的类

+ Creature.java
  + Animal.java
+ MyAnnotation.java
+ MyInterface.java

##### Animal.java

```java
@MyAnnotation
public class Animal extends Creature<String> implements Comparable<String>, MyInterface{

    private String name;
    int age;
    public int id;


    //Constructor
    public Animal() {}

    @MyAnnotation(value = "abc")
    public Animal(String name) {
        this.name = name;
    }

    Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }


    //Methods
    @MyAnnotation
    private String show(String breed) {
        System.out.println("动物品种是" + breed);
        return breed;
    }

    public String display(String sample) throws NullPointerException, ClassNotFoundException {
        return sample;
    }

    private static void showDesc() {
        System.out.println("我是一只可爱的动物");
    }

    //Implement Methods
    @Override
    public int compareTo(String o) {
        return 0;
    }

    @Override
    public void info() {
        System.out.println("这是一只动物");
    }

    @Override
    public String toString() {
        return "Animal{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", id=" + id +
                '}';
    }
}
```

##### Creature.java

```java
import java.io.Serializable;

public class Creature<T> implements Serializable {
    private char gender;
    public double weight;

    private void breath() {
        System.out.println("生物呼吸");
    }

    private void eat() {
        System.out.println("生物吃东西");
    }
}
```

##### MyAnnotaion.java

```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.ElementType.LOCAL_VARIABLE;

@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.RUNTIME)  //如果RUNTIME换成CLASS就不可以在反射中获取了
public @interface MyAnnotation {
    String value() default "hello";
}
```

##### MyInterface.java

```java
public interface MyInterface {
    void info();
}
```