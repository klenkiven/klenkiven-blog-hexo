---
title: CSS 知识简单整理
sidebar:
  - toc
date: 2020-03-17 00:12:19
tags: [前端技术]
categories: [前端技术]
---

## 概述

### 概念

> Cascading Style Sheets  层叠样式表

+ 层叠：多个央视可以作用在同一个html的元素上，同时生效

### 优点

1. 功能强大
2. 将内容的展示和央视的控制分离
   + 降低耦合度
   + 让分工协作更容易
   + 提高开发效率

## CSS的使用

### 方式一：内联样式

+ 在标签内使用Style属性指定CSS代码
+ 不推荐使用，这里耦合性依然很高

```html
<div style="color: red">Hello</div>
```

### 方式二：内部样式

+ 在`<head>`标签内，定义`<style>`标签，`<style>`标签的标签体内容就是CSS代码

+ 示例代码

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>CSS和HTML的结合-方式二</title>
  
      <!-- 内部样式 -->
      <style>
          div {
              color: blue;
          }
      </style>
  
  </head>
  <body>
      <div>Hello css</div>
  </body>
  </html>
  ```

  

### 方式三：外部样式

1. 定义CSS的资源文件
2. 在head标签内，定义link标签，引入外部资源文件

```css
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>CSS和HTML的结合-方式三</title>

    <!-- 外部样式 -->
    <link rel="stylesheet" href="css/a.css" >

</head>
<body>
    <div>Hello css</div>
</body>
</html>

/*a.css文件*/
div {
    color: green;
}
```

### 注意

+ 内联样式、内部样式和外部样式，css作用范围越来越大
+ 内联样式不常用，后期常用内部样式和外部样式

## CSS语法格式

* **格式**

  选择器 {

  ​	/* 这是注释 */

  ​	属性名1:属性值1;

  ​	属性名2:属性值2;

  ​    属性名3:属性值3;

  ​	......

  }

+ **选择器**筛选具有相似特征的元素
+ 注意
  + 每一对属性需要使用分号隔开
  + 最后一对属性可以不加分号

## 选择器

### 基础选择器

1. **id选择器**：选择具体的ID属性值的元素，建议在HTML中ID值唯一
2. **元素选择器**：选择具有相同标签名称的元素
   + ID选择器优先级高于元素选择器
3. **类选择器**：选择具有相同class属性值
   + 类选择器优先级高于元素选择器

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>基础选择器</title>
    <style>
        #IDSTest {
            color: red;
        }
        div {
            color: green;
        }
        .CSTest {
            color: blue;
        }
    </style>
</head>
<body>
    <!-- ID选择器 -->
    <div id="IDSTest">ID选择器测试一</div>
    <div>ID选择器测试二</div>
    <!-- 元素选择器 -->
    <div>元素选择器测试</div>
    <!-- 类选择器 -->
    <p class="CSTest">p标签测试一</p>
    <p class="CSTest">p标签测试二</p>
    <p class="CSTest">p标签测试三</p>
</body> 
</html>
```

### 扩展选择器

1. **选择所有元素**

   语法：* {}

2. **并集选择器**

   语法：选择器1, 选择器2 {}

3. **后代选择器**：筛选选择器1元素下的选择器2元素

   语法：选择器1 选择器2 {}

4. **嫡长子(~~大雾~~)选择器**：筛选选择器1的直接的子元素选择器2

   语法：选择器1 > 选择器2 {}

5. **属性选择器**：选择元素名称，属性名=属性值的元素

   语法：元素名称[ 属性名=属性值 ] {}

6. **伪类选择器**：选择一些元素所具有的状态

   语法：元素: 状态 {}

   如：`<a>` 不同状态

   + link   初始状态
   + over  悬浮状态
   + active  点击状态
   + visited  点击以后状态

## 属性

### 字体，文本

+ font-size：字体大小
+ color：文本颜色
+ text-align：文本的对齐方式
+ line-height：行高

### 背景

+ background：可以使用图片也可以使用纯色
  + 例如：background : url(img/pic.png) no-repeat center; 

### 边框

+ border：设置边框（复合属性）

  用法：border: 边框粗细 边框样式  边框颜色;

### 尺寸

+ height，width：尺寸大小

### 盒子模型

+ margin：外边距  可以设置四个方向上的外边距大小
+ padding：内边距 可以设置四个方向上的内边距大小
  + 默认情况下，内边距会影响盒子的大小
  + box-sizing: border-box  设置盒子的属性，让width和height就是最终盒子的大小
+ float：浮动
  + left  左浮动
  + right  右浮动

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>盒子模型</title>
    <style>
        div {
            border: 1px solid red;
            width: 100px;
        }
        .div1 {
            width: 100px;
            height: 100px;
        }
        .div2 {
            width: 200px;
            height: 200px;
            padding: 50px;
            /*
            设置合资的属性，让width和height就是最终盒子的大小
             */
            box-sizing: border-box;
        }
        .div3 {
            float: left;
        }
        .div4 {
            float: left;
        }.div5 {
            float: right;
        }
    </style>
</head>
<body>
    <div class="div2">
        <div class="div1" ></div>
    </div>

    <div class="div3">aaaaa</div>
    <div class="div4">bbbbb</div>
    <div class="div5">ccccc</div>
</body>
</html>
```

