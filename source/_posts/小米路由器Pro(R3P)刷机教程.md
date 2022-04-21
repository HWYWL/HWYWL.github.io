---
title: 小米路由器Pro(R3P)刷机教程
date: 2022-04-21 14:07:40
tags: 
 - 操作系统
 - Linux
 - OpenWRT
categories: 
 - Linux
keywords: "Linux,路由器"
description: 小米路由器Pro(R3P)刷机教程
---

### 第一步解锁路由器SSH
1.首先需要下载 小米路由器Pro 开发版的ROM固件，地址如下：
```
ROM下载地址：https://bigota.miwifi.com/xiaoqiang/rom/r3/miwifi_r3_firmware_e9f31_2.27.120.bin
```
2.在浏览器输入**192.168.31.1**进入小米管理后台右上角 -> 系统升级 -> 手动升级
![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/20220421141502.png)

选择刚刚下载ROM文件，点击开始升级
![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/20220421141539.png)

3.等待重启

4.解锁ssh
```
解锁工具：https://www.miwifi.com/miwifi_open.html
```

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/1650522046111.png)

按照官网说明进行操作解锁

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/20220421141928.png)

### 第二步连接ssh
使用你喜欢的shell工具连上即可，我喜欢国产的，地址如下：
```
官网：http://www.hostbuf.com/t/988.html
下载地址：http://www.hostbuf.com/downloads/finalshell_install.exe
```
连上ssh
```
地址：192.168.31.1
账号：root
密码：自己下载ssh解锁工具包页面的密码
```

![](https://hwy-figure-bed.oss-cn-hangzhou.aliyuncs.com/blog/image/20220421143355.png)

能看这个界面就算是成功一半了。

### 上传刷机固件
固件下载地址：
```
SCP下载地址：http://www.downza.cn/soft/15245.html
固件下载地址：https://github.com/yeliulee/openwrt-ramips-mt7621-xiaomi_mir3p
```
这是一个大佬做的固件，使用名字为**openwrt-ramips-mt7621-xiaomi_mir3p-squashfs-factory.bin**的固件。
将固件改名为firmware.bin，上传到路由器的 **/tmp** 目录下

在控制台按顺序执行如下命令刷机：
```
cd /tmp
nvram set flag_try_sys1_failed=1 
nvram set flag_try_sys2_failed=0 
nvram set flag_boot_success=0 
nvram commit
dd if=firmware.bin bs=1M count=4 | mtd write - kernel1
mtd erase rootfs0
mtd erase rootfs1
mtd erase overlay
dd if=firmware.bin bs=1M skip=4 | mtd write - rootfs0
reboot
```
刷完不要着急，等个十分钟左右指示灯变蓝（非要着急拔电源变砖了，后果自负），然后进入后台管理即可
**路由器 IP ：192.168.1.1
默认用户名：root
默认密码：password**

### 后续升级
如果发现在网上找到了更好用的固件，想升级怎么办呢？

例如恩山论坛这位大佬的固件，地址：
```
https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=4049182&highlight=R3P
```
**将openwrt-ramips-mt7621-xiaomi_mi-router-3-pro-squashfs-sysupgrade.bin改名为sysupgrade.bin**。
我们只需要下载对应我们路由器的固件下来，按照上面步骤上传到  **/tmp** 目录下，执行如下命令：

```
cd /tmp
sysupgrade -n -v /tmp/sysupgrade.bin
```

等待路由器重启，访问**192.168.1.1**继续操作。

### 从openwrt还原为官方固件
**注意：美光（Micron）闪存的刷回官方固件可能会变砖。**

下载小米原厂固件，最好是开发版固件（请参见上面步骤）并将其重命名为miwifi.bin
在SSH路由器中运行以下命令：
```
fw_setenv flag_try_sys1_failed 0
fw_setenv flag_try_sys2_failed 1
fw_setenv flag_boot_success 0
```
接着：
1.拔电关闭路由器

2.现在，（将您的U盘格式化为FAT / FAT32，如果还不是FAT32），然后将miwifi.bin文件复制到闪存驱动器的根目录（而不是子文件夹）。
将您的U盘连接到路由器，按住重置按钮并打开电源。按住重置按钮，直到黄灯闪烁。等待5分钟，以安装原厂固件。

3.您现在可以登录到192.168.31.1的路由器。

