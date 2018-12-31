---
title: "硬盘安装引导Linux"
categories:
  - Linux
tags:
  - Linux
  - Installation Guide
---

某些时候可能不想使用VPS商默认提供的系统，但是VPS商又不支持上传自定义ISO，这时候可以从硬盘全新安装系统。以下以Debian 9为例，重启后需要到厂商提供的VNC Console操作。

## 将以下文件从 Debian 存档中复制到硬盘中比较方便的地方，比如 /boot/newinstall/

* vmlinuz(内核二进制文件)
* initrd.gz (内存虚拟磁盘映像)

如果您只愿意使用硬盘引导，然后从网络下载其他文件，需要下载[netboot/debian-installer/amd64/initrd.gz](http://ftp.debian.org/debian/dists/stable/main/installer-amd64/current/images/netboot/debian-installer/amd64/) 文件及其对应的内核[netboot/debian-installer/amd64/linux](http://ftp.debian.org/debian/dists/stable/main/installer-amd64/current/images/netboot/debian-installer/amd64/)(vmlinuz)。这将允许您重新分区用于引导的硬盘，需要小心操作。

另一种方法，可以在安装时保持原硬盘的分区不变，您需要下载 hd-media/initrd.gz 文件以及对应的内核，还需要一个 CD（或者 DVD） iso 文件的副本放在硬盘上（确认文件名是以 .iso 结尾的）。这样安装程序就可以从硬盘引导，并从 CD/DVD 映像文件上安装，而不使用网络。

## 配置bootloader,使用GRUB2,在 /boot/grub/ 目录下找到grub.cfg添加

```shell
menuentry 'New Install' {
    insmod part_msdos
    insmod ext2
    set root='(hd0,msdos1)'
    linux /boot/newinstall/vmlinuz
    initrd /boot/newinstall/initrd.gz
}
```

## 重启

重启选择New Install，正常安装。

## 参考文献

* [为从硬盘引导准备文件](https://www.debian.org/releases/stable/amd64/ch04s04.html.zh-cn)
* [在 64-bit PC 上引导安装程序](https://www.debian.org/releases/stable/amd64/ch05s01.html.zh-cn)
