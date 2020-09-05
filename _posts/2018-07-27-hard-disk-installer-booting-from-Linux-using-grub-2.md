---
title: "如何从硬盘引导安装Linux系统"
categories:
  - Linux
tags:
  - 重装Linux系统
  - Debian Installation Guide
---

有时候想安装VPS商不提供的特定版本的Linux系统，或者不方便卸载厂商在系统里集成的一些监控软件，这时候可以自己重装Linux系统。以下以Debian为例，介绍如何从硬盘引导安装Linux系统。

## 准备文件

将以下文件从Debian存档中复制到硬盘中比较方便的地方，比如`/boot/newinstall/`

- vmlinuz（内核二进制文件）
- initrd.gz（内存虚拟磁盘映像）

如果你只愿意使用硬盘引导，然后从网络下载其他文件，需要下载[netboot/debian-installer/amd64/initrd.gz](http://ftp.debian.org/debian/dists/stable/main/installer-amd64/current/images/netboot/debian-installer/amd64/initrd.gz)文件及其对应的内核[netboot/debian-installer/amd64/linux](http://ftp.debian.org/debian/dists/stable/main/installer-amd64/current/images/netboot/debian-installer/amd64/linux)(重命名为vmlinuz)。这将允许你重新分区用于引导的硬盘，需要小心操作。

另一种方法，可以在安装时保持原硬盘的分区不变，你需要下载[hd-media/initrd.gz](http://ftp.debian.org/debian/dists/stable/main/installer-amd64/current/images/hd-media/initrd.gz)文件以及对应的内核[hd-media/vmlinuz](http://ftp.debian.org/debian/dists/stable/main/installer-amd64/current/images/hd-media/vmlinuz)，还需要一个CD（或者DVD）iso文件的副本放在硬盘上（确认文件名是以`.iso`结尾的）。这样安装程序就可以从硬盘引导，并从CD/DVD 映像文件上安装，而不使用网络。

以上文件链接对应Debian官方镜像**最新稳定版**。其它特定版本请查看[Debian全球镜像站](https://www.debian.org/mirror/list.zh-cn.html)，在你下载速度快的镜像站下载。

## 配置bootloader

以使用GRUB2为例，在`/boot/grub/`目录下找到`grub.cfg`文件，添加以下内容

```properties
menuentry 'New Install' {
    insmod part_msdos
    insmod ext2
    set root='(hd0,msdos1)'
    linux /boot/newinstall/vmlinuz
    initrd /boot/newinstall/initrd.gz
}
```

## 重启

1. 在厂商提供的VNC Console操作重启
1. 选择New Install
1. 正常安装

## 参考

- [为从硬盘引导准备文件](https://www.debian.org/releases/stable/amd64/ch04s04.zh-cn.html)
- [在 64-bit PC 上引导安装程序](https://www.debian.org/releases/stable/amd64/ch05s01.zh-cn.html)
