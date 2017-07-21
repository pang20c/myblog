title: lvm 学习总结
date: 2014-11-20 17:02:37
tags:
- linux
category:
- linux
---

LVM(Logical Volume Manager)逻辑卷管理器 理论方面的东西就不写了转一个博客 [理解lvm](http://hily.me/blog/2008/10/understanding-lvm/)

说一下我的理解 lvm要解决的问题就是要把几个离散量打散变成一个大杂烩的连续量，再在这连续量基础上去分成几个我们需要的离散量，

举个分粥的例子 原先就两碗粥怎么分给三个人 办法就是把两碗粥倒回锅里 重新盛出来三碗。。。

好了 我们的原来的两碗粥就是PP （物理分区）且这碗粥不能是串味的 他的文件格式必须是LVM
我们的大锅了 就是VG （卷组）
我们重新盛出来的三碗了 就是 LV （逻辑卷）

这个过程中还有几个概念 PP是不能直接往锅里倒的  他需要先变成PV （物理卷）一个PV是应一个PP的

还有两个概念PE 物理扩展单元（Physical Extends） 和LE 逻辑扩展单元（Logical Extends） 我们可以理解为粥的计量单位 升 只不过PE是用来计量PV 的 LE是计量LV的 PE和LE是一一对应的

下面上点例子
```bash
[root@localhost mytest]#fdisk -l #我事先分好两个新区 [分区教程](http://www.cnblogs.com/gaojun/archive/2012/08/22/2650229.html)
Disk /dev/sdb: 10.7 GB, 10737418240 bytes
255 heads, 63 sectors/track, 1305 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xcbeb1b34

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1               1         262     2104483+  8e  Linux LVM
/dev/sdb2             263        1305     8377897+  8e  Linux LVM
```

将我们的分区PP变为PV
```bash
[root@localhost mytest]# pvcreate /dev/sdb1 
  Physical volume "/dev/sdb1" successfully created
[root@localhost mytest]# pvcreate /dev/sdb2
  Physical volume "/dev/sdb2" successfully created
[root@localhost mytest]# pvdisplay #显示当前的PV有哪些
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               VolGroup
  PV Size               19.51 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              4994
  Free PE               0
  Allocated PE          4994
  PV UUID               P3Lfss-AjZM-Ybeu-PdXN-zWXr-ipn1-sdoDNt

  "/dev/sdb1" is a new physical volume of "2.01 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               2.01 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               7un1fc-rBf6-2vv9-mkuw-m3bd-Fxu5-blDX69

  "/dev/sdb2" is a new physical volume of "7.99 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb2
  VG Name
  PV Size               7.99 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               e2fePJ-Mv1D-Ac6I-p6Kn-kbxg-nWHU-PkMQc0
```

开始把PV往锅里加
可以选择之前已经存在一个VG 用vgextend去扩展它 也可以新建一个VG vgcreate
```bash
[root@localhost mytest]# vgdisplay
  --- Volume group ---
  VG Name               VolGroup
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               19.51 GiB #这是VG扩展前的体积
  PE Size               4.00 MiB
  Total PE              4994
  Alloc PE / Size       4994 / 19.51 GiB
  Free  PE / Size       0 / 0
  VG UUID               Jf633Y-moeI-est9-MHP5-yVSf-0SOz-skQhzZ
[root@localhost mytest]# vgextend VolGroup /dev/sdb1
  Volume group "VolGroup" successfully extended
[root@localhost mytest]# vgextend VolGroup /dev/sdb2
  Volume group "VolGroup" successfully extended
[root@localhost mytest]# vgdisplay
  --- Volume group ---
  VG Name               VolGroup
  System ID
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               29.50 GiB #可以看到VG的体积变大了
  PE Size               4.00 MiB
  Total PE              7552
  Alloc PE / Size       4994 / 19.51 GiB
  Free  PE / Size       2558 / 9.99 GiB
  VG UUID               Jf633Y-moeI-est9-MHP5-yVSf-0SOz-skQhzZ
```

下面是开始往外盛粥了 去创建3个LV
```bash
[root@localhost mytest]# lvcreate -L 3G -n newLv1 VolGroup
  Logical volume "newLv1" created
[root@localhost mytest]# lvcreate -L 3G -n newLv2 VolGroup
  Logical volume "newLv2" created
[root@localhost mytest]# lvcreate -L 3G -n newLv3 VolGroup
  Logical volume "newLv3" created

[root@localhost mytest]# mkfs.ext3 /dev/VolGroup/newLv1 #格式化我们刚分出来的分区
[root@localhost mytest]# mkdir -p /data/mnt1
[root@localhost mytest]# mount /dev/VolGroup/newLv1 /data/mnt1 #把我们的分区挂载到一个目录

[root@localhost mytest]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root   18G   12G  4.6G  73% /
tmpfs                         495M     0  495M   0% /dev/shm
/dev/sda1                     485M   57M  403M  13% /boot
/dev/mapper/VolGroup-newLv1   3.0G   69M  2.8G   3% /data/mnt1
```
记得写入/etc/fstab 在重启后自动挂载 关于fstab的配置 [https://wiki.archlinux.org/index.php/Fstab_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29](https://wiki.archlinux.org/index.php/Fstab_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29)

下面我们试一下删除一个逻辑卷 和扩容一个逻辑卷
```bash
[root@localhost mytest]# lvremove -f VolGroup/newLv2 #删除一个未挂载的逻辑卷
  Logical volume "newLv2" successfully removed
```
在改变逻辑卷有个要注意的地方 我们的文件系统是建立在逻辑卷（分区）之上的
![文件系统和分区](/img/lvm.jpg)
我们通过df看到的文件夹的大小 实际是文件系统的大小 ，文件系统的大小 是可以小于分区的大小的
当我们想扩大一个文件系统的时候 首先要扩展它的分区（逻辑卷）然后再扩展文件系统 不然盒子就撑爆了
而当我们想缩小一个文件系统的时候 相反的首先要缩小文件系统 再缩小分区 不然 里面的盒子就压烂了
如果我们想扩容 那可以在线扩容 而如果是要缩小 则需要先umount 这个分区 缩小完再挂回去


* 扩大一个逻辑卷
```bash
[root@localhost ~]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root   18G   12G  4.6G  73% /
tmpfs                         495M     0  495M   0% /dev/shm
/dev/sda1                     485M   57M  403M  13% /boot
/dev/mapper/VolGroup-newLv1   3.0G   69M  2.8G   3% /data/mnt1
[root@localhost ~]# lvextend -L +1G /dev/VolGroup/newLv1  #扩展逻辑卷
  Extending logical volume newLv1 to 4.00 GiB
  Logical volume newLv1 successfully resized
[root@localhost ~]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root   18G   12G  4.6G  73% /
tmpfs                         495M     0  495M   0% /dev/shm
/dev/sda1                     485M   57M  403M  13% /boot
/dev/mapper/VolGroup-newLv1   3.0G   69M  2.8G   3% /data/mnt1 #我们只扩大了逻辑卷但是并没有扩大文件系统所以 df看到的大小并没变
[root@localhost ~]# resize2fs /dev/VolGroup/newLv1 #通过resize2fs 让文件系统重新适应它所在分区的大小
resize2fs 1.41.12 (17-May-2010)
Filesystem at /dev/VolGroup/newLv1 is mounted on /data/mnt1; on-line resizing required
old desc_blocks = 1, new_desc_blocks = 1
Performing an on-line resize of /dev/VolGroup/newLv1 to 1048576 (4k) blocks.
The filesystem on /dev/VolGroup/newLv1 is now 1048576 blocks long.

[root@localhost ~]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root   18G   12G  4.6G  73% /
tmpfs                         495M     0  495M   0% /dev/shm
/dev/sda1                     485M   57M  403M  13% /boot
/dev/mapper/VolGroup-newLv1   4.0G   71M  3.7G   2% /data/mnt1
```
* 缩小一个逻辑卷
[一篇教程 http://seriousbirder.com/blogs/lvreduce-ext4-example/](http://seriousbirder.com/blogs/lvreduce-ext4-example/)

```bash
[root@localhost ~]# umount /data/mnt1/
[root@localhost ~]# e2fsck -f /dev/VolGroup/newLv1 #先检测一下文件系统
e2fsck 1.41.12 (17-May-2010)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
/lost+found not found.  Create<y>? yes

Pass 4: Checking reference counts
Pass 5: Checking group summary information

/dev/VolGroup/newLv1: ***** FILE SYSTEM WAS MODIFIED *****
/dev/VolGroup/newLv1: 13/262144 files (0.0% non-contiguous), 34397/1048576 blocks
[root@localhost ~]# resize2fs -p /dev/VolGroup/newLv1 2G #调整文件系统的大小
resize2fs 1.41.12 (17-May-2010)
Resizing the filesystem on /dev/VolGroup/newLv1 to 524288 (4k) blocks.
Begin pass 3 (max = 32)
Scanning inode table          XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
The filesystem on /dev/VolGroup/newLv1 is now 524288 blocks long.
[root@localhost ~]# lvreduce -L 2G /dev/VolGroup/newLv1 #调整逻辑卷的大小
  WARNING: Reducing active logical volume to 2.00 GiB
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce newLv1? [y/n]: Y
  Reducing logical volume newLv1 to 2.00 GiB
  Logical volume newLv1 successfully resized
[root@localhost ~]# mount /dev/VolGroup/newLv1 /data/mnt1/ #重新挂载回去
[root@localhost mnt1]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root   18G   12G  4.6G  73% /
tmpfs                         495M     0  495M   0% /dev/shm
/dev/sda1                     485M   57M  403M  13% /boot
/dev/mapper/VolGroup-newLv1   2.0G   69M  1.9G   4% /data/mnt1
```
还有为啥一个逻辑卷要起两个名字 我不理解 欢迎指教
```bash
[root@localhost mnt1]# ll /dev/mapper/VolGroup-newLv1 /dev/VolGroup/newLv1
lrwxrwxrwx. 1 root root 7 Nov 22 00:12 /dev/mapper/VolGroup-newLv1 -> ../dm-2
lrwxrwxrwx. 1 root root 7 Nov 22 00:12 /dev/VolGroup/newLv1 -> ../dm-2
```


