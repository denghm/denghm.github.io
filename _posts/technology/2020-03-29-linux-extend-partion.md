---
layout: article
title: "Linux下扩展磁盘分区容量"
modified: 2020-03-29 
author: Voyager
toc: false
comments: true
categories: technology
summary: Linux经常会遇到磁盘空间不足的问题，庆幸的是，新一代的文件系统提供了可以扩展空间的能力。
---

Linux经常会遇到磁盘空间不足的问题，庆幸的是，新一代的文件系统提供了可以扩展空间的能力。我们可以随时添加新的硬盘扩展已有文件系统的存储能力。虚拟机的预分配的磁盘空间用尽后扩展的磁盘也需要通过这种方式调整磁盘空间。群晖NAS更换大容量的磁盘后，默认增加的磁盘空间时新的挂载卷，如果需要扩展旧有卷的空间。也需要进行手工调整，


### 1.格式化剩余磁盘空间为新的磁盘

[root@localhost]# fdisk /dev/sda

分别键入以下参数：

```shell
p    查看已分区数量（我看到有两个 /dev/sda1 和/dev/sda2）
n    新增加一个分区
p    分区类型，选择主分区   
          分区号选3（1和2已占用，见上）     
          默认选择（起始扇区）回车     
          默认选择（结束扇区）
t    修改分区类型       
          选分区38e     修改为LVM（8e就是LVM）
w    写分区表，完成后退出fdisk命令
```



### 2. 使用partprobe重新读取分区表或者重启机器。

```shell
partprobe
```

### 3. 初始化刚才的分区

```shell
pvcreate /dev/sda3
```

** 不需要创建xfs文件系统 mkfs.xfs -f /dev/sda3，
这一步会创建 ，否则会出现以下错误

```
WARNING: xfs signature detected on /dev/sda3 at offset 0. Wipe it? [y/n]: y Wiping xfs signature on /dev/sda3.
```

### 4.将初始化过的分区加入到虚拟卷组

```shell
vgextend centos /dev/sda3
```

### 5.扩展已有卷的容量

```
lvextend -l +100%FREE /dev/mapper/centos-root
```

### 6.在线扩容同步文件系统

对不同的文件系统，不同的操作方式

xfs文件系统   

```
 xfs_growfs /dev/mapper/centos-rootext2 
```

ext3 ext4 文件系统    

```
resize2fs 
```

btrfs文件系统 

```
btrfs filesystem resize max /home
```

