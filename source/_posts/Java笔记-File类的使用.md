---
title: Java笔记---File类的使用
sidebar:
  - toc
date: 2020-02-29 18:29:10
tags: [Java基础知识]
categories: [Java基础知识]
---

1. File类的一个对象，代表一个**文件**，或者一个**文件夹**
2. File类声明在java.io包下面
3. File类中涉及关于文件和文件目录的创建，删除，重命名，修改时间，文件大小等方法，并未涉及到写入或读取文件内容的操作。如果需要写入或读入文件内容，则需要IO流来完成。
4. 后续File类的对象常会作为参数传入到流构造器中，指明读取或写入的”终点“。

## 如何创建一个File类的实例

#### 构造器一：public File(String pathname)

```java
public void instantFile() {
   //构造器一：public File(String pathname)
   //这里使用的是相对路径，相对于Module
   File file1 = new File("hello.txt");
   //还可以使用绝对路径
   File file2 = new File("D:/KlenKiven/IDEA/VideoCoding/hello.txt");
    
}
```

#### 构造器二：public File(String parent, String child)

```java
File file = new File("C:\\Directory", "hello.txt");
```

#### 构造器三：public File(File parent, String child)

```java
File file1 = new File("C:\\Directory");
File file2 = new File(file1, "hello.txt");
```

## File类的常用方法

#### 适用于所有文件的方法

+ String getAbsolutePath()  : 获取绝对路径
+ String getPath()  : 获取路径
+ String getParent()  : 获取父级路径
+ String getName() :  获取文件名
+ long length()  : 获取文件长度(字节数)
+ long lastModified()  : 获取最后一次修改的时间，返回的是一个毫秒数

```java
public void fileMethod() {
    File file1 = new File("hello.txt");
    File file = new File("D:\\KlenKiven\\File\\hello.txt");
    System.out.println(file1.getAbsolutePath());
    System.out.println(file.getPath());
    System.out.println(file.getName());
    System.out.println(file.getParent());
    //文件内容为“Hello World”
    System.out.println(file.length());
    System.out.println(new Date(file.lastModified()));
}
```

结果输出如下：

> D:\KlenKiven\IDEA\VideoCoding\hello.txt
> D:\KlenKiven\File\hello.txt
> hello.txt
> D:\KlenKiven\File
> 11
> Fri Feb 28 15:11:35 CST 2020

#### 适用于文件目录的方法

+ String[] list()  : 获取指定目录下的所有文件和文件目录的名字的数组
+ File[] listFiles()  : 获取指定目录下的所有文件或者文件目录的File数组

```java
public void fileMethod() {
	File dir = new File("D:/KlenKiven");

	String[] list = dir.list();
	for(String temp: list)
        //可以输出文件名
	    System.out.println(temp);

	File[] listF = dir.listFiles();
	for(File temp: listF)
        //以下可以输出文件名以及文件最后的修改时间
	    System.out.println(temp.getName() + "\t" + new Date(temp.lastModified()));
}
```

输出结果略。

#### 文件重命名为指定的文件路径

> public boolean renameTo(File dest)
>
> 把文件重命名为指定的文件路径

例如：file1.renameTo(file2);

**注意**：

​	要想保证renameTo()返回值是真的， file2在硬盘中不可以存在。

​	完成以后，原来的文件消失，重命名的文件出现。

```java
public void fileMethod() {
	File file5 = new File("D:\\KlenKiven\\File\\hello.txt");
	File file6 = new File("D:\\KlenKiven\\hi.txt");

	boolean renameTo = file5.renameTo(file6);
	System.out.println(renameTo);
}
```

#### File类中的一些判断功能

这些方法名，见名知意

```java
@Test
public void judgement() {
    File file1 = new File("D:\\KlenKiven\\File");
    File file2 = new File(file1, "hello.txt");
    File file3 = new File(file1, "hi.txt");

    //判断是否是一个文件夹
    System.out.println(file1.isDirectory());
    //判断是不是一个文件
    System.out.println(file1.isFile());
    //判断文件是否存在
    System.out.println(file1.exists());
    //判断是否可读
    System.out.println(file1.canRead());
    //判断是否可写
    System.out.println(file1.canWrite());
    //判断是否是隐藏文件
    System.out.println(file1.isHidden());
}
```

#### File类的创建和删除功能

+ `public boolean creatNewFile()` 创建**文件**，若文件存在则不创建，并返回flase
+ `public boolean mkdir()` 创建**文件目录**，如果文件目录存在，就不创建了。但是不创建上层目录。
+ `public boolean mkdirs()` 创建文件目录，如果上层目录不在，那么上层目录和这个目录一起创建。
+ `public boolean delete()` 删除文件或者文件目录
