---
title: ArchLinux 2019.11.01安装流程--安装基本系统
sidebar:
  - toc
date: 2019-11-28 22:54:00
tags: [Linux]
categories: [Linux]
---

## 安装前的一些话
本文是参考官方文档ArchLinux的[Installation guide(简体中文)](https://wiki.archlinux.org/index.php/Installation_guide_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29)加实际操作编写的。
1. 有啥都好说，**转载时请注明作者**，这是**基本素质**，也是**法律要求**
2. 安装是在**虚拟机**上进行安装的，如果想要在**实体机**上安装，请认真参考官方文档进行安装。
3. 本机器安装的时候官网下载的版本是Archlinux2019.11.01版本的，其他**更加新**的版本，本文有些地方可能不适用，一切以**官方文档**为准。
4. 本文中的命令如果没有特殊说明都是在**root**权限下进行的
5. 如果文章有啥问题，欢迎大佬指正！我也是个小菜鸡，斗胆放出教程。
6. 如果有志同道合的朋友有问题，可以在文章下方评论！PS.建议先看**官方文档**和**百度**尝试解决，如果解决不了，可以评论一起讨论。
## 安装前的一些准备活动
### 1. 验证启动模式
```bash
ls /sys/firmware/efi/efi
```
虚拟机这里是UEFI启动模式
### 2.连接到因特网
由于是虚拟机，采用的是**有线连接**。

```bash
dhcpcd                #自动获取IP地址
ping www.baidu.com    #测试网络是否连通
```

### 2.更新系统时间
使用命令 `timedatectl` 确保时间正确

```bash
timedatectl set-ntp true
```
### 3.建立硬盘分区
1. 这里使用 `cfdisk` 命令进行分区操作，这个是一个图形化的分区界面傻瓜化，很好用。
![建立分区表](https://img-blog.csdnimg.cn/20191122235039402.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4NDc4NjEz#pic_center,size_16,color_FFFFFF,t_70 ) 
这里选择 gpt 分区表比较合适于 UEFI 启动方式，以下是**UEFI 的推荐分区方案：**

挂载点	| 分区	| 分区类型	| 建议大小
---|---|---|---
/mnt/boot or /mnt/efi |	/dev/sdX1|	EFI分区	|260–512 MiB
/mnt	/dev/sdX2|	Linux x86-64 |根目录 (/)	|剩余空间
[SWAP]	|/dev/sdX3	|Linux swap (交换空间)	|大于 512 MiB

2. **Enter**回车选择**New**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123000157402.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4NDc4NjEz#pic_center,size_16,color_FFFFFF,t_70 )
3. 在该位置输入分区大小[单位为M, G(MB, GB)]
![输入分区大小](https://img-blog.csdnimg.cn/20191123000420563.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4NDc4NjEz#pic_center,size_16,color_FFFFFF,t_70 )
4. 然后可以看到，已经分好了一个分区
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123000836661.png#pic_center )
5. 创建一个 **Swap 分区**，如法炮制
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123001054835.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4NDc4NjEz#pic_center,size_16,color_FFFFFF,t_70 )
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123001332717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4NDc4NjEz#pic_center,size_16,color_FFFFFF,t_70)
6. 最后的一个分区直接，选择free space然后一路回车就好了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123001514358.png#pic_center )
7. 记一下每个分区叫啥、大小，以后有用，最后记得**保存**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112300243628.png#pic_center )
### 4.格式化分区
1. 格式化**普通分区**
	```bash
	mkfs.fat -F32 /dev/sda1      #把sda1格式化为fat32并挂载到 /boot/efi
	mkfs.ext4 /dev/sda3          #挂载到 /
	```
2. 格式化 **Swap 分区**
	```bash
	mkswap /dev/sda2         #这个是swap分区
	```
### 5.挂载分区
```bash
mount /dev/sda3 /mnt
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```
## 正式开始安装
开头的一切都是值得的，现在开始了正式的安装步骤，这里将会有大片等待时间，可以喝口水稍作休息。
### 1.选择镜像
文件 `/etc/pacman.d/mirrorlist` 定义了软件包会从哪个 **镜像源** 下载。这里使用VIM进行文件修改。
对 VIM 有了解的可以按照下面步骤操作，不会的有简单方法。
```bash
vim /etc/pacman.d/mirrorlist
```

> 下面的操作极其考验视力

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112300512640.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4NDc4NjEz#pic_center,size_16,color_FFFFFF,t_70)
选定后，直接击打键盘`dd`剪切这一行，然后使光标上移到顶部，按大写`P`粘贴这一行。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123005516653.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4NDc4NjEz#pic_center,size_16,color_FFFFFF,t_70)
然后，按`Esc`退出**编辑模式**。最后`:wq`保存。

**简单**方法：
使用`nano`命令
```bash
nano /etc/pacman.d/mirrorlist
```
这个是图形化界面，直接在顶行输入
`Server = http://mirrors.cqu.edu.cn/archlinux/$repo/os/$arch`
退出时 使用`Ctrl + X`退出。
### 2.安装必须的软件包
```bash
pacstrap /mnt base base-devel linux linux-firmware
```
接下来就是**逼格超级高**的刷屏幕等待时间
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123011128574.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4NDc4NjEz#pic_center,size_16,color_FFFFFF,t_70)
**无报错**表示安装软件包安装成功了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191123011921830.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4NDc4NjEz#pic_center,size_16,color_FFFFFF,t_70)
## 配置系统
这一部分也是很繁琐的一些配置步骤，不过如果能理解这些，对linux的理解会有一些提高的。
### 1.Fstab
用以下命令生成`fstab`文件
```bash
genfstab -U /mnt >> /mnt/etc/fstab   #生成fstab文件
cat /mnt/etc/fstab                   #检验是否正确生成fstab文件
```
### 2.Chroot
这一步进入新安装的系统进行进一步的配置
```bash
arch-chroot /mnt
```
### 3.时区
将时间设置为**Shanghai时间**
```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
运行`hwclock`以生成 `/etc/adjtime`：
```bash
hwclock --systohc
```
### 4.本地化

> 本地化的程序与库若要本地化文本，都依赖 **Locale**，后者明确规定地域、货币、时区日期的格式、字符排列方式和其他本地化标准等等。在下面两个文件设置：`locale.gen` 与 `locale.conf`。
`/etc/locale.gen` 是一个仅包含注释文档的文本文件。指定您需要的本地化类型，只需**移除**对应行前面的注释符号（＃）即可，建议选择带 UTF-8 的项

```bash
nano /etc/locale.gen
```

> 说明一下，我安装这些包的时候，好像`nano`并未正常安装
> 因此我需要安装一些常用的软件包
```bash
pacman -S nano 
```

这里直接简单的使用`nano`编辑文档

> 删除下列行之前的`#`号
>en_US.UTF-8 UTF-8
>zh_CN.UTF-8 UTF-8
>zh_TW.UTF-8 UTF-8

接下来执行`locale-gen`生成 local 信息
```bash
locale-gen
```
创建 `locale.conf` 并编辑 `LANG` 这一 变量

> 将这里设置为 `en_US.UTF-8`，系统就会用英文显示。如果这里设置为 `zh_CN.UTF-8`会出现乱码问题。

```bash
nano /etc/locale.conf   #输入 LANG=en_US.UTF-8 保存
```
### 5.网络
创建`/etc/hostname`文件
```bash
nano /etc/hostname    #输入 myhostname 保存
nano /etc/hosts
```
>在 **hosts** 文件中添加以上内容：
> 	127.0.0.1	localhost
>	::1		localhost
>	127.0.1.1	myhostname.localdomain	myhostname

### 6.Root 密码
**设置 ROOT 密码**极其重要！密码千万不要忘记！！！
```bash
passwd
```
然后按照提示设置密码！
### 7.安装引导程序(UEFI这部分也比较麻烦)
这里就显示出，前面的**分区工作**是极其重要的
1. **要安装`grub`软件**：
	```bash
	pacman -S grub efibootmgr
	```
2. **安装grub** `grub-install`
	>  其中 `GRUB`是启动引导器[^3]
	>  `efibootmgr` 被 `GRUB` 脚本用来将启动项写入 `NVRAM`。
	> 这里挂载 [ESP]		(https://wiki.archlinux.org/index.php/EFI_system_partition#Mount_the_partition) 到 `/boot/efi` 因此创建以下 文件夹 

	```boot
	mkdir -p /boot/efi/EFI
	grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
	```
3. 配置Grub `grub-mkconfig`
	```bash
	grub-mkconfig -o /boot/grub/grub.cfg
	```
	> 这里多说几句，如果想要识别多系统，需要安装`os-prober`

## 退出Chroot并重启
```bash
exit    #退出 Chroot
reboot  #重启系统
```
## 小结
至此，Archlinux的**基本系统**已经安装好了！接下来就一些桌面安装，环境安装之类的个性化安装了！
如果有时间，我的**桌面安装**和**环境配置**将会在后续放出