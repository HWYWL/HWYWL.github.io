---
title: Ubuntu18.04修改swap分区大小
date: 2020-02-26 13:21:37
tags: 
 - 操作系统
 - Linux
categories: 
 - Linux
keywords: "Linux,分区"
description: 在虚拟机装大数据组件的时候，经常会遇到内存不足的的情况，所以使用交换分区代替内存使用。
cover: https://s2.ax1x.com/2020/02/26/3NDpcR.png
---
## 查看初始状态
查看原先swap大小

```
root@gpu-2:~# free -h
              总计         已用        空闲      共享    缓冲/缓存    可用
内存：         62G        417M         38G        3.0M         23G         61G
交换：        2.0G          0B        2.0G
root@gpu-2:~#
```
原先swap文件位置

```
root@gpu-2:~# swapon  -s
文件名             类型      大小       已用  权限
/swapfile1         file      12582908    0   -2
root@gpu-2:~#
```

## 创建一个新的swap文件
一般swap分区要大于或等于物理内存(1-1.5倍)，最大一般有20G即可，我这里创建12G：

```
root@gpu-2:~# cd /
root@gpu-2:/# dd if=/dev/zero of=/swapfile1 bs=1G count=12
记录了12+0 的读入
记录了12+0 的写出
12884901888 bytes (13 GB, 12 GiB) copied, 17.1497 s, 751 MB/s
root@gpu-2:/# ll
总用量 14680188
drwxr-xr-x  25 root root        4096 4月  10 17:22 ./
drwxr-xr-x  25 root root        4096 4月  10 17:22 ../
drwxr-xr-x   2 root root        4096 4月  10 06:59 bin/
drwxr-xr-x   4 root root        4096 4月  10 06:59 boot/
drwxrwxr-x   2 root root        4096 3月  25 20:44 cdrom/
drwxr-xr-x   5 netc netc          58 4月   9 14:01 data/
drwxr-xr-x  19 root root        4340 3月  29 07:50 dev/
drwxr-xr-x 127 root root       12288 4月  10 06:59 etc/
drwxr-xr-x   3 root root        4096 3月  25 20:45 home/
lrwxrwxrwx   1 root root          33 4月   3 06:24 initrd.img -> boot/initrd.img-4.18.0-17-generic
lrwxrwxrwx   1 root root          33 4月   4 06:31 initrd.img.old -> boot/initrd.img-4.18.0-16-generic
drwxr-xr-x  21 root root        4096 3月  25 20:56 lib/
drwxr-xr-x   2 root root        4096 2月  10 08:12 lib64/
drwx------   2 root root       16384 3月  25 20:41 lost+found/
drwxr-xr-x   2 root root        4096 2月  10 08:12 media/
drwxr-xr-x   2 root root        4096 2月  10 08:12 mnt/
drwxr-xr-x   2 root root        4096 2月  10 08:12 opt/
dr-xr-xr-x 323 root root           0 3月  26 10:39 proc/
drwx------   8 root root        4096 3月  26 11:22 root/
drwxr-xr-x  31 root root        1040 4月  10 17:20 run/
drwxr-xr-x   2 root root       12288 4月  10 06:58 sbin/
drwxr-xr-x  12 root root        4096 3月  26 08:54 snap/
drwxr-xr-x   2 root root        4096 2月  10 08:12 srv/
-rw-------   1 root root  2147483648 3月  25 20:41 swapfile     # 之前的swap文件
-rw-r--r--   1 root root 12884901888 4月  10 17:22 swapfile1    # 新创建的swap文件
dr-xr-xr-x  13 root root           0 4月  10 17:11 sys/
drwxrwxrwt  10 root root       12288 4月  10 17:23 tmp/
drwxr-xr-x  10 root root        4096 2月  10 08:12 usr/
drwxr-xr-x  14 root root        4096 2月  10 08:20 var/
lrwxrwxrwx   1 root root          30 4月   3 06:24 vmlinuz -> boot/vmlinuz-4.18.0-17-generic
lrwxrwxrwx   1 root root          30 4月   4 06:31 vmlinuz.old -> boot/vmlinuz-4.18.0-16-generic
root@gpu-2:/#
```

## 创建swap文件系统

```
root@gpu-2:/# mkswap -f swapfile1
mkswap: swapfile1：不安全的权限 0644，建议使用 0600。
正在设置交换空间版本 1，大小 = 12 GiB (12884897792  个字节)
无标签， UUID=3779f693-8356-42e9-8a2c-2ab51f12654a
root@gpu-2:/# chmod 0600 swapfile1
root@gpu-2:/# ll
总用量 14680188
drwxr-xr-x  25 root root        4096 4月  10 17:22 ./
drwxr-xr-x  25 root root        4096 4月  10 17:22 ../
drwxr-xr-x   2 root root        4096 4月  10 06:59 bin/
drwxr-xr-x   4 root root        4096 4月  10 06:59 boot/
drwxrwxr-x   2 root root        4096 3月  25 20:44 cdrom/
drwxr-xr-x   5 netc netc          58 4月   9 14:01 data/
drwxr-xr-x  19 root root        4340 3月  29 07:50 dev/
drwxr-xr-x 127 root root       12288 4月  10 06:59 etc/
drwxr-xr-x   3 root root        4096 3月  25 20:45 home/
lrwxrwxrwx   1 root root          33 4月   3 06:24 initrd.img -> boot/initrd.img-4.18.0-17-generic
lrwxrwxrwx   1 root root          33 4月   4 06:31 initrd.img.old -> boot/initrd.img-4.18.0-16-generic
drwxr-xr-x  21 root root        4096 3月  25 20:56 lib/
drwxr-xr-x   2 root root        4096 2月  10 08:12 lib64/
drwx------   2 root root       16384 3月  25 20:41 lost+found/
drwxr-xr-x   2 root root        4096 2月  10 08:12 media/
drwxr-xr-x   2 root root        4096 2月  10 08:12 mnt/
drwxr-xr-x   2 root root        4096 2月  10 08:12 opt/
dr-xr-xr-x 323 root root           0 3月  26 10:39 proc/
drwx------   8 root root        4096 3月  26 11:22 root/
drwxr-xr-x  31 root root        1040 4月  10 17:20 run/
drwxr-xr-x   2 root root       12288 4月  10 06:58 sbin/
drwxr-xr-x  12 root root        4096 3月  26 08:54 snap/
drwxr-xr-x   2 root root        4096 2月  10 08:12 srv/
-rw-------   1 root root  2147483648 3月  25 20:41 swapfile
-rw-------   1 root root 12884901888 4月  10 17:22 swapfile1
dr-xr-xr-x  13 root root           0 4月  10 17:11 sys/
drwxrwxrwt  10 root root       12288 4月  10 17:23 tmp/
drwxr-xr-x  10 root root        4096 2月  10 08:12 usr/
drwxr-xr-x  14 root root        4096 2月  10 08:20 var/
lrwxrwxrwx   1 root root          30 4月   3 06:24 vmlinuz -> boot/vmlinuz-4.18.0-17-generic
lrwxrwxrwx   1 root root          30 4月   4 06:31 vmlinuz.old -> boot/vmlinuz-4.18.0-16-generic
root@gpu-2:/#
```

## 开启新的swap

```
root@gpu-2:/# swapoff /swapfile
root@gpu-2:/# free -h
              总计         已用        空闲      共享    缓冲/缓存    可用
内存：         62G        417M         38G        3.0M         23G         61G
交换：          0B          0B          0B
root@gpu-2:/# swapon /swapfile1
root@gpu-2:/# free -h
              总计         已用        空闲      共享    缓冲/缓存    可用
内存：         62G        420M         38G        3.0M         23G         61G
交换：         11G          0B         11G
root@gpu-2:/#
```

## 设置开机启动

```
root@gpu-2:/# vim /etc/fstab
/swapfile                                 none            swap    sw              0       0
改为
/swapfile1                                none            swap    sw              0       0
root@gpu-2:/#
```

## 重启
重启，然后查看是否有问题

```
shutdown -r now
```
可以先在虚拟机上测试，如果成功的话，再在物理机操作，如果都没问题的话，可以删掉旧的swap文件

```
rm -f /swapfile
```
