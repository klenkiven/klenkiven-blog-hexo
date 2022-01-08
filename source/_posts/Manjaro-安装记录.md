---
title: Manjaro 安装记录
sidebar:
  - toc
date: 2020-03-27 21:43:06
tags: [Linux]
categories: [Linux]
---

## 写在之前

**不建议没有任何备份之前在实体机上安装！！！**

**不建议没有任何备份之前在实体机上安装！！！**

**不建议没有任何备份之前在实体机上安装！！！**

## Manjaro概述

Manjaro是Archlinux的一个衍生版，Archliunx的高度可定制性，颇受linux吧大佬的喜爱。Manjaro继承了Archlinux的所有优点，还免去了Archlinux的安装繁琐的弊端。

## 镜像烤录

镜像烤录的话，可以使用Rufus刻录U盘。过程的话一般都很简单，这里不做赘述。镜像的下载可以在[清华大学开源镜像站](https://mirrors.tuna.tsinghua.edu.cn/)来下载相关的镜像。

## 系统安装

这个的话，开机选择BIOS，选择U盘启动啥的。下面给出一些教程：

+ [不同品牌电脑进入BIOS的方法集合](https://jingyan.baidu.com/article/0320e2c11bfad81b87507b11.html)
+ [关闭Secure Boot](https://jingyan.baidu.com/article/6079ad0ecf605028fe86db74.html)
+ [BIOS中设置U盘启动](https://jingyan.baidu.com/article/fea4511ad31699f7bb912501.html)

做完这些以后，就是进入LiveCD了。

在官方文档中，有详细的安装指南：[Manjaro WIki](https://wiki.manjaro.org/index.php/Main_Page)可以参考。

### LiveCD启动时的注意

LiveCD启动时，一般都有一些选项，应该选择正确的语言  lang => 中文 => zh_CN，一般的话driver选项选择non-free会安装闭源驱动，可以解决一些类似Nvidia独立显卡驱动的问题。

### 关于手动分区的建议

一般来说，在虚拟机上面，直接全盘安装就好了。虚拟机上面一般都是MBR的分区表。在实体机上面，一般都是UEFI启动了，需要GPT分区表，并且在安装的时候，需要划出一部分区域，作为EFI分区。

| 分区挂载位置 | 分区文件系统 | 标记 |
| :----------: | :----------: | :--: |
|      /       |     ext4     |     |
|    /home     |     ext4     |       |
| /boot/efi | fat32 | boot |

我按装得时候遇到的问题是，先手动分区/home，/boot/efi，结果屡屡报错，所以手动分区的时候，一定要先分出 挂载根目录的分区。

### Windows和Linux共存时，时间不一致的问题

[知乎回答：解决linux与Windows时间不一致问题](https://www.zhihu.com/question/46525639/answer/157272414)

## 安装好以后的配置

### 输入法配置

一般来说，linux安装好以后是没有中文输入法的。

+ [Archlinux+gnome安装中文输入法](https://www.cnblogs.com/huangyingting/p/10599404.html)
  + 配置archlinuxcn源
  + 安装输入法本体

### gnome桌面配置神器 -- gnome-tweaks

`sudo pacman -S gnome-tweaks`

+ 这个软件可以设置gnome桌面全居缩放
+ 可以管理gnome桌面的插件
+ 等等等等

### 配置ZSH

zsh相当好用，比bash好用多了，其中oh-my-zsh项目还有很多zsh的皮肤。安装系统时候自带了zsh了，因此只要配置好就行了。

+ 把当前用户的shell设置为zsh

  + `sudo gedit /etc/passwd`里面配置用户的shell

    找到自己的用户名

    ![passwd文件](https://images2017.cnblogs.com/blog/1298835/201712/1298835-20171220152607256-1842050776.png)

    把黄色的字/bin/bash，改为/bin/zsh，即可。

+ [OhMyZsh，安装，主题配置方法](https://www.jianshu.com/p/0f3dcec21a97)

### 安装Google Chrome

自带的是FireFox浏览器，其实也不错，但是我更喜欢用Chrome。Chrome浏览器在AUR仓库里面，安装的时候需要先安装yay，来安装AUR仓库里面的软件。

#### 方法一：AUR仓库

+ 安装yay

  `sudo pacman -S yay`

+ 安装google-chrome

  `yay -S google-chrome`

#### 方法二：中文仓库

上文已经在安装输入法那篇博客哪里配置了，aechlinuxcn仓库，这里面就有chrome浏览器了。直接`sudo pacman -S google-chrome`即可。

### 安装JDK

对于学习java，或者是java相关的开发，就需要安装JDK。Oracle java的话安装配置太麻烦不如直接openjdk安装。

+ 安装JDK8（全套）
  + `sudo pacman -S jdk8-openjdk` 
  + `sudo pacman -S openjdk8-src`
  + `sudo pacman -S openjdk8-doc`
+ 安装JDK10, 11也是同理

### 安装QQ，TIM

+ 安装QQ

  `sudo pacman -S deepin.com.qq.im`

+ 安装TIM

  `sudo pacman -S deepin.com.qq.office`

### 安装网易云音乐

`sudo pacman -S netease-cloud-music`

### 其他的软件啥的一般只到名字就行，直接pacman就可
