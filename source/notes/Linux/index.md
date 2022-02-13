---
robots: noindex,nofollow
sitemap: false
menu_id: notes
layout: wiki
wiki: 便笺
seo_title: 🐧Linux常用配置
order: 302
---

## Archlinux的安装 && Manjaro的安装

{% post_link ArchLinux-2019-11-01安装流程-安装基本系统 %}

{% post_link Manjaro-安装记录 %}

## 文件权限设置篇

1. 将所有的文件类型的文件，设置为644权限
   ```bash
   find KlenKiven/ -type f -exec chmod 644 {} \;
   ```
2. 将所有的目录类型的文件设置为755权限
   ```bash
   find KlenKiven/ -type d -exec chmod 755 {} \;
   ```

## 只要涉及到网络全都可以代理篇

1. Git设置代理 `clone`
   ```bash
   # 设置全局的代理
   git config --global http.proxy socks5://127.0.0.1:1080
   # 取消全局的代理
   git config --global --unset http.proxy
   ```
2. Curl下载东西也可以设置代理下载
   ```bash
   curl [balabalaba] -x http://127.0.0.1:8889
   ```

## 常用的Gnome插件

`Tray Icons Reloaded`：系统托盘

`Dynamic Panel Transparency`：动态透明

`Arch Linux Updates Indicator`：Archlinux必备

`Espresso`：自动挂起自动熄屏控制（一杯咖啡）

## 关于Deepin的那些事

### 安装Deepin之类的软件遇到的问题

安装Deepin之类的软件很依赖32位的库什么的，所以安装Deepin之类的软件的时候，必须把 `nultilib` 这个repository打开。原因就不解释了，下面说的很清楚。直接取消注释就行

```bash
# If you want to run 32 bit applications on your x86_64 system,
# enable the multilib repositories as required here.

#[multilib-testing]
#Include = /etc/pacman.d/mirrorlist

[multilib]
Include = /etc/pacman.d/mirrorlist
```

参考资料：[Official repositories - ArchWiki - multilib](https://wiki.archlinux.org/index.php/Official_repositories#multilib)

### Deepin QQ和TIM无法自动加载图片的问题

#### 方法一：IPV6的关闭

1. 首先，执行命令 `ip addr` 找到相关的网卡名字，例如，wlp0s20f3，可以看到相关的信息.
    > 我已经关闭了IPV6了，不然应该会有 inet6 的字眼
    >

    ```
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 巴拉巴拉
    2: wlp0s20f3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
        link/ether 50:e0:85:c1:e0:b2 brd ff:ff:ff:ff:ff:ff
        inet 192.168.1.103/24 brd 192.168.1.255 scope global dynamic noprefixroute wlp0s20f3
           valid_lft 6604sec preferred_lft 6604sec
        inet6 巴拉巴拉
    6: enp0s20f0u3u3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
        link/ether 00:e0:4c:36:00:25 brd ff:ff:ff:ff:ff:ff
    ```
2. 编辑配置文件 `sudo vim /etc/sysctl.conf`
    * 这里是禁止特定的网卡的IPV6的配置，比如禁止 lo, wlp0s20f3 网卡的IPV6功能 ( 推荐 )

    ```
    net.ipv6.conf.lo.disable_ipv6 = 1
    net.ipv6.conf.wlp0s20f3.disable_ipv6 = 1
    ```

    * 下面是全局禁用IPV6的方式
      ```
      net.ipv6.conf.all.disable_ipv6 = 1
      net.ipv6.conf.default.disable_ipv6 = 1
      ```
3. 执行配置文件 `sudo sysctl -p`
4. 然后执行 `ip addr` 就可以检查是否禁用成功了。

#### 方法二：设置代理

在Manjaro下面deepin的QQ和TIM无法打开相关设置，我这里就不再过多探究这个问题了。

### KDE：Deepin QQ和TIM无法在KDE桌面启动的问题

1. 安装`gnome-setting-daemon`
    ```bash
    sudo pacman -S gnome-settings-deamon
    ```
2. 执行下面的命令
    ```bash
    cp /etc/xdg/autostart/org.gnome.SettingsDaemon.XSettings.desktop ~/.config/autostart/ 
    ```
3. 设置KDE开机启动
    系统设置 => 开机和关机 => 自动启动
    然后，勾选GNOME Setting Deamon's xsetting plugin
4. 如果你是高分屏幕就需要设置dpi
    ```bash
    gsettings set org.gnome.desktop.interface text-scaling-factor 1.31
    ```

    > 我这里设置的是1.27，你也可以设置成更大或者更小的数，数字越大，放大倍数越大mari
    >

## KDE：MySQL Workbench

MySQL Workbench在archlinux中出现 Could not store password: The name org.freedesktop.secrets was not provided by any .service files的错误
解决方案是安装 gnome-keyring 包。

## Sogoupinyin: Error parsing key “overrides”

```bash
sudo nano /usr/share/glib-2.0/schemas/50_sogoupinyin.gschema.override
	# change the line of overrides：
	overrides=Gtk/IMModule:<fcitx>
	# change to:
	overrides=Gtk/IMModule:<fcitx>
sudo glib-compile-schemas /usr/share/glib-2.0/schemas/
reboot
```

# Archlinux 安装的一些东西

## Fcitx5 相当不错的一个输入法

### 安装及其配置

```bash
sudo pacman -S fcitx5-im				# Fcitx5的本体
sudo pacman -S fcitx5-chinese-addons	# Fcitx5的中文扩展支持
sudo pacman -S fcitx5-material-color	# Fcitx5的一个主题
```

完成安装步骤以后，对其进行相关的配置。

```bash
# 首先要配置的是自动启动
cp /usr/share/applications/fcitx5.desktop ~/.config/autostart-scripts/
# 接下來配置相关的环境变量配置
vim .xprofile
	# 文件添加下面的内容
	export GTK_IM_MODULE=fcitx5
	export QT_IM_MODULE=fcitx5
	export XMODIFIERS="@im=fcitx5
```

### 自己遇到的问题：候选框字体设置

KDE桌面（**安装了KCM**）的解决办法：

`设置=>区域设置=>输入法配置=>配置附加组件=>经典用户恶极面=>设置`

非KDE桌面：

右键系统托盘上面的输入法图标，选择`配置`，然后`配置附加组件=>经典用户恶极面=>设置`

## GRUB美化

### 下载主题文件

> 登入网站Gnome-Look
>
> https://www.gnome-look.org/
>

找到GRUB Theme标签，就可以找自己喜欢的主题啦！我喜欢的主题是Tela主题。

### 具体安装

```bash
# 解压文件
tar -Jxf Tela-1080p.tar.xz
# 将文件移动到指定位置
sudo mkdir /usr/share/grub/themes/Tela
sudo cp -rf ~/Downloads/Tela-1080p/*  /usr/share/grub/themes/Tela
# 
```

# 其他问题

### modprobe: FATAL: Module vboxnetflt not found in directory /lib/modules/5.8.5-arch1-1

安装 `linux-header` , 然后执行  `sudo modprobe vboxdrv vboxnetflt vboxnetadp vboxpci `

## 激活Thin Windows

一、汉化方法：

1. 将中文语言包lp.cab放到C盘的根目录
2. 以管理员身份运行CMD命令输入：
    ```cmd
    dism  /online  /add-package  /packagepath:C:\lp.cab
    ```
3. 安装完成后，依次运行下面命令，重启电脑即可：
    ```cmd
    bcdedit /set {current} locale zh-cn
    bcdboot %WinDir% /l zh-cn
    ```

二、系统激活
 证书下载地址：链接: [https://pan.baidu.com/s/1nv86wIl](https://links.jianshu.com/go?to=https%3A%2F%2Fpan.baidu.com%2Fs%2F1nv86wIl) 密码: nfu1
 1、将证书文件放到C盘的根目录
 2、以管理员身份运行CMD命令，依次运行下面命令即可：

    `cmd     slmgr.vbs /ilc c:\pkeyconfig-embedded.xrm-ms     slmgr.vbs /ilc c:\Security-SPP-Component-SKU-Embedded-VLBA-ul.xrm-ms     slmgr.vbs /ilc c:\Security-SPP-Component-SKU-Embedded-VLBA-ul-oob.xrm-ms     slmgr.vbs /ipk XGY72-BRBBT-FF8MH-2GG8H-W7KCW     slmgr.vbs /xpr     `

## 电脑有声音，耳机没声音

https://linux.cn/article-3489-1.html


