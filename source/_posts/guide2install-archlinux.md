title: ArchLinux折腾笔记
toc: true
date: 2016-05-22 14:57:13
categories: Linux
thumbnail: /images/archlinux-logo.png
banner: /images/archlinux-logo.png
tags: [Linux, 折腾]
---
前天把虚拟机里的ubuntu升级到16.04的时候挂了，就不想再折腾Ubuntu了，一直听说Arch Linux的wiki很完整，日常遇到的问题都能完整的wiki，这一点比其他linux的发行版好的太多了，如果要说困难的地方，应该就是安装比其他发行版难，好在有完整的wiki，开始折腾。

<!--more-->
![archlinux-logo](/images/archlinux-logo.png)

环境：macbook pro 13（2014mid），parallel desktop， archlinux-2016.05.01-dual.iso


#### 分区

使用cgdisk分区[wiki](https://wiki.archlinux.org/index.php/Gdisk#Gdisk_usage_summary)：

```
# lsblk
# cgdisk /dev/sda
```

分三个区，linux filesystem的hexcode是8300，swap分区的代码是8200，最后效果如下，write后退出：
![](/images/cgdisk.png)

#### 格式化挂载分区

格式化：
```
# mkfs.ext4 /dev/sda1
# mkfs.ext4 /dev/sda3
# mkswap /dev/sda2
# swapon /dev/sda2
```
挂载：
```
# mount /dev/sda1 /mnt
# mkdir /mnt/home
# mount /dev/sda3 /mnt/home
```
linux下什么都没有表示正确执行，如果有提示说明有错，重新执行或者google错误原因找到解决问题。

#### 软件源
编辑/etc/pacman.d/mirrorlist,添加国内源，保存退出：
```
# vi /etc/pacman.d/mirrorlist

Server = http://mirrors.163.com/archlinux/$repo/os/$arch
Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch

```

#### 安装基本系统：
```
# pacstrap /mnt base base-devel
```

#### 配置基本系统：
生成fstab：
```
genfstab /mnt >> /mnt/etc/fstab

```
chroot到新安装的基本系统中，chroot就是进入某个目录，并设置root密码：
```
# arch-chroot /mnt
# passwd
```
设置主机名称
```
# echo arch-linux > /etc/hostname
```
设置时钟
```
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
然后用vim打开/etc/locale.gen，然后找到以下四行，取消注释:
```
en_US.UTF-8
zh_CN.UTF-8
zh_CN.GBK
zh_CN.GB2312
```
运行# locale-gen,再编辑/etc/locale.conf，里面写上:
```
LANG="en_US.UTF-8"
```
如果这样系统还会提示$LC_*H或者Lang之类的错误的时候，请在终端里输入：

```
exprot LC_ALL = C

```


建立初始ramdisk环境:

```
# mkinitcpio -p linux
```

启用DHCP服务：
```
# systemctl enable dhcpcd.service
```
引导器配置：
```
# pacman -S gptfdisk
# pacman -S syslinux
# syslinux-install_update -i -a -m
# vi /boot/syslinux/syslinux.cfg
```
將 root=/dev/sda3 改成 root=/dev/sda1
![syslinux.cfg](/images/AL017.gif)

完成重启：
```
# exit
# umount -R /mnt
# reboot
```
现在应该可以进入新系统了

####配置桌面

添加用户：
```
# useradd -m raighne
# passwd raighne
```

装X Server
```
# pacman -S xorg xorg-xinit xterm xorg-xeyes xorg-xclock
```

装yaourt,vi /etc/pacman.conf:
```
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```
然后

```
# pacman -Sy archlinuxcn-keyring yaourt
```

安装Gnome：
```
# pacman -S gnome
# systemctl enable gdm
# systemctl start gdm
```
第一行是安装gnome，第二行是让gdm开机自启动，第三行是现在就启动它

这样安装工作基本已经完成，还有一些按个人喜欢的配置，比如安装输入法，安装编辑器之类的，因为是在虚拟机里安装的，我还要再安装下parallel tools，折腾到底为止吧。
