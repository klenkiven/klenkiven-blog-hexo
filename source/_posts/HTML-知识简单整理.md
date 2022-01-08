---
title: HTML 知识简单整理
sidebar:
  - toc
date: 2020-03-17 00:13:19
tags: [前端技术]
categories: [前端技术]
---

## 概念

> HTML (**Hyper Text Markup Language** **超文本标记语言**)  是最基础的网页开发语言

+ **超文本**：超文本是用*超链接* 的方法，将各种不同的空间文字信息组织在一起的网状文本。
+ **标记语言**：由*标签* 构成的语言。<标签名称> 如 html，xml
  + 标记语言不是编程语言

## 快速入门

### 语法

1. html文档的**后缀名**是`.html` 或 `.htm` 两者含义完全相同
2. **标签**分类
   + 围堵标签：有开始标签和结束标签 `<html> </html>`
   + 自闭和标签：开始标签和结束标签在一起 `<br/>`
3. **标签可以嵌套**：需要正确嵌套，不可以你中有我，我中有你。
4. 再开始标签中可以定义属性。属性是由键值对构成，值需要用引号（单双引号都可以）
5. html的标签不区分大小写，但是建议使用小写

### 示例代码

```html
<html>
    <head>
        <title>这是Title</title>
    </head>

    <body>
        <font color="red">Hello World</font> <br/>
        <font color="green">Hello World</font>
    </body>
</html>
```

## HTML标签 -- 文件标签

> 构成html最基本的标签

+ `<html>` 

  html文档的根标签

+ `<head>`

  头标签。用于指定html文档的一些属性。引入外部的资源

+ `<title>`

  标题标签。

+ `<body>`

  体标签。

+ `<!DOCTYPE html>`

  定义文档类型

## HTML标签 -- 文本标签

> 和文本有关的标签

+ `<h1> to <h6>`：标题标签
  + 后面的数字代表不同等级的标签
  + 随着数字的增大，字体逐渐减小
+ `<p>`：段落标签，段落后自动加一个换行
+ `<br>`：换行标签
+ `<hr>`：水平线标签   *h5不支持*
  + color：颜色属性
  + width：宽度
  + size：粗细
  + align：对齐方式 left right center
+ `<b>`：字体加粗
+ `<i>`：斜体
+ `<font>`：字体标签  *h5已经不支持*
  + color：颜色
  + size：大小
  + face：字体样式
+ `<center>` *h5已经不支持* 使得在center标签中的标签元素居中
+ 属性定义：
  + color：
    + red  green  blue
    + rgb(val1, val2, val3) 值的取值范围0~255
    + #ffffff
  + width：
    + 数值：width = "20"  数值单位默认是px像素
    + 数值%：表示相对于父元素的比例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文本标签</title>
</head>
<body>
    <!-- 注释内容 -->
    <!-- br 换行 -->
    白日依山尽<br/>
    黄河入海流<br>

    <!-- 标题标签 h1~h6 -->
    这是一个标题<br/>
    <h1>这是一个标题</h1>
    <h2>这是一个标题</h2>
    <h3>这是一个标题</h3>
    <h4>这是一个标题</h4>
    <h5>这是一个标题</h5>
    <h6>这是一个标题</h6>

    <!-- 段落标签 <p> -->
    1 这是一段话，这是一段话，这是一段话，这是一段话。<br/>
    2 这是一段话，这是一段话，这是一段话，这是一段话。<br/>
    3 这是一段话，这是一段话，这是一段话，这是一段话。<br/>
    4 这是一段话，这是一段话，这是一段话，这是一段话。<br/>
    <p>1 这是一段话，这是一段话，这是一段话，这是一段话。</p>
    <p>2 这是一段话，这是一段话，这是一段话，这是一段话。</p>
    <p>3 这是一段话，这是一段话，这是一段话，这是一段话。</p>
    <p>4 这是一段话，这是一段话，这是一段话，这是一段话。</p>

    <!-- 显示水平线 hr -->
    <hr color="blue" size="5" align="left" width="200"/>

    <!--  加粗 斜体 <b> <i> -->
    正常样式<br/>
    <b>加粗</b>---------<i>斜体</i><br/>

    <!-- 字体标签 <font> -->
    <font color="red" size="5" face="楷体">字体标签测试</font>

    <!-- 展示一张图片 img -->
    <img src="imgs/pic.png"/>
</body>
</html>
```



## HTML标签 -- 图片标签

> 用于展示图片

`<img>` 用于展示图片，是一个自闭和标签。

+ 属性参数：
  + **src**：显示图像的URL
    + 使用**相对路径**，如"./image/pic.png"，"../WEB-2/image/pic.png"，其中一个点，代表当前目录，两个点代表上一级目录。
  + **alt**：如果图片加载失败，会显示alt的值
  + align：图片的位置 

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文本标签</title>
</head>
<body>
    <!-- 展示一张图片 img -->
    <img src="imgs/pic.png" alt="这是一张图片"/>
</body>
</html>
```



## HTML标签 -- 列表标签

+ 有序列表
  + `<ol>` order list 有序列表
    + type：可以选择多种属性：数字，a，A，i，I
    + start：从几开始
  + `<li>` 代表列表的每一列
+ 无序列表
  + `<ul>` unorder list 无序列表
    + type：*h5已经不再支持* 规定列表项目富豪的样式
      + disc 原点  square 方形  circle  圆圈
  + `<li>` 代表列表的每一列

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>列表标签</title>
</head>
<body>
    <!-- 有序列表 ol -->
    起床要做的事情：<br/>
    <ol type="A" start="5">
        <li>睁眼</li>
        <li>看手机</li>
        <li>穿衣服</li>
        <li>洗漱</li>
    </ol><br/>

    <!-- 无序列表 ul -->
    起床要做的事情：<br/>
    <ul type="disc" start="5">
        <li>睁眼</li>
        <li>看手机</li>
        <li>穿衣服</li>
        <li>洗漱</li>
    </ul><br/>
    <ul type="square" start="5">
        <li>睁眼</li>
        <li>看手机</li>
        <li>穿衣服</li>
        <li>洗漱</li>
    </ul><br/>
    <ul type="circle" start="5">
        <li>睁眼</li>
        <li>看手机</li>
        <li>穿衣服</li>
        <li>洗漱</li>
    </ul><br/>
</body>
</html>
```



## HTML标签 -- 链接标签

`<a>` 定义一个超链接。

+ 属性参数
  + href：指定访问资源的URL（统一资源定位符），也可以指定当前项目内部的资源，可以插入邮件地址使用“mailto:xxx@mail.com”
  + target：_self 默认值在当前页面打开 _blank 在空白页面打开

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>超链接标签</title>
</head>
<body>
    <!-- 链接标签 <a> -->
    <a href="https://www.baidu.com/">点我</a>
    <br/>

    <a href="https://www.baidu.com/" target="_blank">打开新的选项卡</a>
    <br/>

    <a href="mailto:xxx@mail.com">联系我们</a>
    <br/>

    <!-- 实现点击图片跳转链接的效果 -->
    <a href="2-TextLable.html"><img src="imgs/pic.png"/></a>
    <br/>
</body>
</html>
```

## HTML标签 -- 块标签

`<span>` 本身并没有任何的效果，文本信息在一行展示，也叫做行内标签，内联标签

`<div>` 和span相比，div没有任何样式效果

+ 每个div可以占满一行

这两个标签没有任何的样式效果，可以使用css一起配合使用。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>块标签</title>
</head>
<body>
    <!-- div span -->
    <span>span测试1</span>
    <span>span测试2</span>
    <hr/>
    <div>div测试1</div>
    <div>div测试2</div>
</body>
</html>
```

## HTML标签 -- 语义化标签

> html5中为了提高程序的可读性，提供了一些标签。这些标签本身不会有任何的样式，配合CSS使用

如：

+ `<header><header/>`

+ `<footer><footer/>`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>语义化标签</title>
</head>
<body>
    <!-- 头部标签 -->
    <header>

    </header>

    <!-- 尾部标签 -->
    <footer>

    </footer>
</body>
</html>
```

## HTML标签 -- 表格标签

`<table><table/>` ：定义表格

+ width：宽度
+ border：边框
+ cellpading：定义内容和单元格的距离
+ cellspace：定义单元格之间的距离，如果指定为0，那么表格之间的距离为0
+ bgcolor：背景颜色

`<tr><tr/>` ：定义行

`<td><td/>` ：定义单元格

+ colspan：合并列
+ rowspan：合并行

`<th><th/>` ：定义表头单元格

`<caption>` ：定义表格标题

下面三个没有任何的效果，需要配合CSS使用：

> 注意！如果合并行列的时候，行处于一下不同的标签中时，是不可以被合并的。比如一个单元格在`<thead>`，另一个在`<tbody>`那么这两个单元格不可以被合并

+ `<thead>`：表示表的头部

+ `<tbody>`：表示表的主题

+ `<tfoot>`：表示表的尾部

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>表格标签</title>
</head>
<body>
    <!-- 表格标签 -->
    <table border="2" width="50%" cellpadding="0" cellspacing="0">
        <caption>学生成绩信息表</caption>
        <thead>
        <tr>
            <th>编号</th>
            <th>姓名</th>
            <th>成绩</th>
        </tr>
        </thead>

        <tbody>
        <tr>
            <td>1</td>
            <td>张三</td>
            <td>60</td>
        </tr>
        </tbody>

        <tfoot>
        <tr>
            <td>2</td>
            <td>李四</td>
            <td>90</td>
        </tr>
        </tfoot>
    </table>

    <hr/>

    <table border="2" width="50%" cellpadding="0" cellspacing="0">
        <caption>学生成绩信息表</caption>
        <tr>
            <th rowspan="2">编号</th>
            <th>姓名</th>
            <th>成绩</th>
        </tr>

        <tr>
            <td>张三</td>
            <td>60</td>
        </tr>

        <tr>
            <td>2</td>
            <td colspan="2">李四</td>
        </tr>
    </table>
</body>
</html>
```

## HTML标签 -- 表单标签

### 表单

+ 概念：用于采集用户输入的数据

+ `<form>`：用于定于表单的，可以**定义一个范围**，范围代表采集用户数据的范围
  + 属性：
    + action：指定提交属性数据的URL
    + method：指定**提交方式**（一共7种，两种比较常用）
      + get：（**默认值**）
        1. 请求参数**会**在地址栏中显示，会封装到请求行中
        2. 请求参数的长度**有限制**
        3. get请求不太安全
      + post：
        1. 请求参数**不会**在地址栏中显示，会封装到**请求体**中
        2. 请求参数的长度**没有限制**
        3. post请求较为安全
  + 表单中的数据（表单项）要想被提交，**必须指定**其name属性。

### 表单项

+ `<input>`：可以通过改编type属性值，改编元素展示的样式

  + type属性：
    + text：文本输入框

      1. placeholder：指定输入框的提示信息，当输入框的内容发生变化，会自动清空提示内容
      2. value：设定默认值

    + password：密码输入框

    + radio：单选框

      1. 要想实现单选，需要设置相同的name值 

      2. 一般会给每一个单选框日供一个value属性，指定其被选中后提交的值
      3. checked属性，可以指定默认值

    + checkbox：复选框

    + file：文件选择框

    + hidden：隐藏域

    + submit：提交按钮

    + button：普通按钮

    + image：图片提交按钮

      + src：指定图片的路径

    + color：取色器

    + date：选择时间

    + datetime-local：带有小时分钟的时间

    + email：邮箱输入器，并且自带正则表达检测

+ `<lable>`：指定输入项的文字描述信息

  + lable的for属性会和input的id属性值对应，如果对应了，会让input输入框获取焦点。

+ `<select>`：下拉列表 

  + 结构：

    ```html
    <select>
        <option></option>
        <option></option>
        <option></option>
    </select>
    ```

+ `<textarea>`：文本域

  + cols：指定列数
  + rows：指定行数

### 示例代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>表单标签</title>
</head>
<body>
    <!--
        form  用于定于表单的，可以定义一个范围，范围代表采集用户数据的范围
     -->
    <form action="#" method="get">
        <label for="username"> 用户名 </label>：<input type="text" name="username" placeholder="请输入用户名" id="username"><br/>
        密码：<input type="password" name="password" placeholder="请输入密码"><br/>

        性别：<input type="radio" name="gender" value="male" checked>男
        <input type="radio" name="gender" value="female">女<br/>

        爱好：<input type="checkbox" name="hobby" value="shpping">逛街
        <input type="checkbox" name="hobby" value="java" checked>Java
        <input type="checkbox" name="hobby" value="game">游戏<br/>

        上传头像：<input type="file" name="file"><br/>

        隐藏域：<input type="hidden" name="hide" value="aaa"><br/>
        取色器：<input type="color" name="color"><br/>
        生日：<input type="date" name="brithday"><br/>
        生日：<input type="datetime-local" name="brithday"><br/>
        邮箱：<input type="email" name="email"><br/>
        年龄：<input type="number" name="age"><br/>

        省份：
        <select name="province">
            <option>-----请选择-----</option>
            <option value="1">北京</option>
            <option value="2">上海</option>
            <option value="3">山西</option>
        </select><br/>
        自我描述：<textarea cols="50" rows="5"></textarea>

        <input type="button" name="button" value="这是一个按钮">
        <input type="image" src="imgs/pic.png">
        <input type="submit" value="登录">
    </form>
</body>
</html>
```
