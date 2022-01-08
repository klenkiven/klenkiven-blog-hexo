---
title: JavaScript知识简单整理
sidebar:
  - toc
date: 2020-08-18 17:49:19
tags: [前端技术]
categories: [前端技术]
---

## 概述

### 概念：一门客户端脚本语言

+ 运行在客户端浏览器中的。每一个浏览器都有JavaScript的解析引擎
+ 脚本语言：不需要编译，直接就可以被浏览器解析执行了

### 功能

+ 可以来增强用户的html页面的交互过程，可以控制html元素，让页面有一些动态效果，增强用户的体验

### JavaScript = ECMAScript + BOM + DOM

## ECMAScript：客户端脚本语言的标准

### 基本语法

#### 与html结合方式

+ 内部JS
  + 定义`<script>`标签，标签体内容就是js代码
+ 外部JS
  + 定义`<script>`标签，通过src属性引入外部的JS文件
+ 注意：
  + 可以写在HTML的任意位置
  + 放置的顺序会影响JS的执行顺序
  + `<script>`标签可以定义多个

#### 注释

+ 单行注释：//注释内容
+ 多行注释：/* 多行注释 */

#### 数据类型

+ 原始数据类型（基本数据类型）：
  + **number**：整数 / 小数 / NaN
  + **string**：字符串类型
  + **boolean**：布尔类型
  + **null**：一个对象为空的占位符
  + **undefined**：未定义。如果一个变量没有给初始化值，则会被认为是undefined
+ 引用数据类型：对象

#### 变量

+ 概念：一块存储数据的内存空间
+ Java语言是强类型的语言，而JavaScript是弱类型语言
+ 语法：var 变量名 =  初始化值;  //初始化值可以是任意值
+ typeof运算符：要检查的变量是什么样的数据类型
  + 注意：null原酸后得到的是object

#### 运算符

+ 一元运算符：只有一个运算数的运算符
  + ++，--，+(正号), -(负号)
  + 在JS中，如果运算数不是运算符所要求的类型，那么JS引擎会自动的讲运算符进行类型转换
    + 其他类型转number
      + string转number：按照字面之转换，如果字面值不是数字，则转换成NaN
+ 算术运算符：
  + +，-，*，/，% ...
+ 赋值运算符：
  + =，+=，-= ...
+ 比较运算符：
  + <, >, ==, ===(全等于), <=, >=
+ 逻辑运算符：
  + &&, ||, ! 
+ 三元运算符：
  + ?  : 
+ 特殊语法：
  + 语句以分号结尾，如果一行只有一条语句，则分号可以省略
  + 声明变量，不用加var也可以
    + 加var定义的是局部变量
    + 不加var定义的是全局变量（不建议使用）

#### 流程控制语句

+ if ... else
+ switch:
  + 在Java中switch可以接受的数据类型：byte, int, shor, char, 枚举, 字符串
  + 在JS中可以接受任何值
+ while
+ do ... while
+ for

#### 练习：99乘法表

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>九九乘法表</title>
    <style>
        td {
            border: 3px solid black;
        }
    </style>
    <script>
        document.write("<table align='center'>");
        for (var j = 1; j <= 9; j++){
            document.write("<tr>")
            for (var k = 1; k <= j; k++) {
                //输出
                document.write("<td>"+j+"*"+k+"="+j*k+"</td>");
            }
            document.write("</tr>")
        }
        document.write("</table>");
    </script>
</head>
<body>

</body>
</html>
```

### 基本的对象

+ **Function对象**：描述一个函数对象

  + **创建**：

    + 方式一

      var fun = new Function(形参列表, 方法体);

    + 方式二

      function 方法名称(形参列表) { 方法体 }

    + 方式三

      var 方法名 = function(形参列表) { 方法体 }

  + **方法**：

  + **属性**：

    + length：形参的个数

  + **特点**： 

    + 方法定义时，形参的类型不用写，返回值类型也可不写

    + 方法是一个对象，如果定义名称相同的方法，会**覆盖**

    + 在JS中，方法的调用只与方法的名称有关和参数列表无关

    + 在方法声明中，有一个**隐藏的内置对象**（数组），arguments，封装所有的实际参数，类似于Java中的可变长参数。

      ```javascript
      function fuc() {
          /* 查看隐藏参数的长度 */
          alert(arguments.length);
      }
      
      //求一个函数所有参数的和
      function sum() {
          var result = 0;
          for(int i = 0; i < arguments.length; i++){
              if(typeof(arguments[i]) == "number")
              	result += arguments[i];
          }
          return result;
      }
      ```

  + **调用**：

    + 方法名称(实际参数列表)

+ **Array对象**：数组对象

  + 创建：
    + var arr = new Array(元素列表);
    + var arr = new Array(默认长度);
    + var arr = [元素列表];
  + 方法
  + 属性
  + 特点
    1. JS中数组的元素的类型是可变的

+ **JS中的自定义对象**

  + 对象的定义

    ```js
    var obj = new Object();  // 对象实例（空对象）
    obj.Attr = value;        // 定义一个属性
    obj.func = function (){}  // 定义一个函数
    ```

  + 对象的访问

    ```js
    alert(obj.Attr);
    alert(obj.fuc());
    ```

+ **花括号形式的自定义对象**

  ```js
  var obj = {
      Attr1: value1,       // 定义一个属性
      Attr2: value2,       // 定义一个属性
      func: function() {}  // 定义一个函数
  } // 对象实例
  
  ```

### JS中的事件

+ onload：加载完成事件，页面加载完成之后，常用与做页面JS代码的初始化操作
+ onclick：单击事件，常用按钮的点击响应事件
+ onblur：失去焦点事件，常用语输入框失去焦点后验证其输入内容是否合法
+ onchange：内容发生改变事件，常用于下拉列表和输入框内容发生改变后的操作
+ onsubmit：表单提交事件，常用于表单提交前，验证所有表单项是否合法

### 事件注册

#### 事件的注册

告诉浏览器，当时间响应后要执行那些代码，叫做事件注册或者事件绑定

+ 静态注册事件：通过html标签的属性直接赋予时间响应后的代码，这种方式我们称为静态注册

+ 动态注册事件：事先通过JS代码得到标签的DOM对象，然后再通过DOM对象.事件名=function(){} 这种形式赋予事件

  动态注册基本步骤：

  1. 获取标签对象
  2. 标签对象事件名 = function(){}

#### onload事件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8"\>
    <title>OnloadEvent</title>
    <script type="text/javascript">
        //onload事件的方法
        function onloadFun() {
            alert('静态注册onclick事件');
        }

        //onload动态注册
        window.onload = function () {
            alert('动态注册onclick事件');
        }
    </script>
</head>
<!-- 静态注册onload事件 -->
<!-- <body onload="onloadFun()"> -->
<!-- 动态注册onload注册 -->
<body >
</body>
</html>
```

#### onclick事件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8"\>
    <title>OnloadEvent</title>
    <script type="text/javascript">
        //静态注册
        function onclickFun() {
            alert('静态注册onclick事件');
        }
        //动态注册
        window.onload = function () {
            // 1. 获取标签对象
            /*
            document 是 javascript语言的一个对象（文档）
            */
            var btnobj = document.getElementById("button2");
            // 2. 通过标签独享.事件名 = function(){}
            btnobj.onclick = function () {
                alert('动态注册onclick事件');
            }
        }
    </script>
</head>

<body>
    <button onclick="onclickFun()">botton1</button>
    <button id="button2">botton2</button>
</body>
```

#### onblur事件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8"\>
    <title>OnblurEvent</title>
    <script type="text/javascript">
    // 静态注册
    function onblurFunc() {
        // console对象是控制台对象，有JS提供的对象
        // 专门用于控制台打印
        console.log("静态注册onblur事件");
    }

    // 动态注册
    window.onload = function () {
        var textobj = document.getElementById("password");
        textobj.onblur = function () {
            console.log("动态注册onblur事件");
        }
    }
    </script>
</head>

<body>
    用户名：<input type="text" onblur="onblurFunc()"/><br>
    密码：<input type="text" id="password"/><br>

</body>
</html>
```

#### onchange事件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8"\>
    <title>OnchangeEvent</title>
    <script type="text/javascript">
    // 静态注册
    function onchangeFunc() {
        alert("静态注册onchange事件");
    }

    // 动态注册
    window.onload = function () {
        var textobj = document.getElementById("select");
        textobj.onchange = function () {
            alert("动态注册onchange事件");
        }
    }
    </script>
</head>

<body>
    请选择你的英雄：
    <select onchange="onchangeFunc()">
        <option>A1</option>
        <option>A2</option>
        <option>A3</option>
        <option>A4</option>
    </select>

    请选择你的英雄：
    <select id="select">
        <option>B1</option>
        <option>B2</option>
        <option>B3</option>
        <option>B4</option>
    </select>

</body>
</html>
```

#### onsubmit事件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8"\>
    <title>OnsubmitEvent</title>
    <script type="text/javascript">
    // submit时，要验证所有的表单是否正确
    // 如果有一个表单不合法便会阻止提交
    // 静态注册
    function onsubmitFunc() {
        alert("静态注册Onsubmit事件 -- 发现不合法");
        return false;
    }

    // 动态注册
    window.onload = function () {
        var textobj = document.getElementById("form");
        textobj.onsubmit = function () {
            alert("动态注册Onsubmit事件 -- 发现不合法");
            return false;
        }
    }
    </script>
</head>

<body>
    <form action="http://localhost:8080" method="GET" onsubmit="return onsubmitFunc();">
        <input type="submit" value="静态注册"/>
    </form>

    <form action="http://localhost:8080" method="GET" id="form">
        <input type="submit" value="动态注册"/>
    </form>
</body>
</html>
```

## DOM模型

> DOM全称是 Document Object Model 文档对象模型
>
> 吧文档中的标签，属性，文本转换为对象来管理。                                 

DOM模型，体现在 `document` 对象。Document文档，由一个树状结构保存。 

1. Document管理了所有的HTML对象内容
2. document他是一种树状结构的文档，有层级关系。
3. 把所有的标签都对象化
4. 可以通过document访问所有的标签对象

### Document对象的方法介绍

`document.getElementById(elementId)`

+ 通过标签的**ID属性**查找标签dom对象，elementId是标签的ID属性值

`document.getElementByName(elementName)`

+ 通过标签的**name属性**查找标签的dom对象，elementName标签的name属性值

`document.getElementByTagName(tagname)`

+ 通过**标签名**查找标签dom对象，tagname是标签名

`document.createElement(tagName)`

+ 通过给定的**标签名**。**创建一个标签对象**，tagName是要创建的标签名

## 节点的常用属性和方法

> 节点就是标签对象

### 方法

+  `getElementByTagName(tagname)` 方法

  通过具体的元素节点调用该方法，可以回去当前节点的制定标签名子节点

+ `appendChild(oChildNode)` 方法

  增加一个子节点，oChildNode是要增加的子节点

### 属性

+ childNodes：当前节点的所有子节点
+ firstChild
+ lastChild
+ parentNode：当前节点的父节点
+ nextSibling：当前节点的下一个节点
+ previousSibling：当前节点的上一个节点
+ className：获取或设置标签的 class 属性值
+ innerHTML：获取/设置其实标签和恶结束标签中的内容
+ innerText：获取/设置其实标签和恶结束标签中的文本
