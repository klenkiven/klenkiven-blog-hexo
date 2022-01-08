---
title: KDE：Deepin QQ和TIM无法在KDE桌面启动的问题
sidebar:
  - toc
date: 2020-04-21 10:41:30
tags: [Linux]
categories: [Linux]
---

1. 安装`gnome-setting-daemon`

   ```bash
   sudo pacman -S gnome-setting-deamon
   ```

2. 执行下面的命令

   ```bash
   cp /etc/xdg/autostart/org.gnome.SettingsDaemon.XSettings.desktop ~/.config/autostart
   ```

3. 设置KDE开机启动

   系统设置 => 开机和关机 => 自动启动

   然后，勾选GNOME Setting Deamon's xsetting plugin

4. 如果你是高分屏幕就需要设置dpi

   ```bash
   gsettings set org.gnome.desktop.interface text-scaling-factor 1.25
   ```

   > 我这里设置的是1.25，你也可以设置成更大或者更小的数，数字越大，放大倍数越大