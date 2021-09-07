烧录镜像
==
1.连接TF卡到Ubuntu虚机，查看分区情况
--
```
lazy@ubuntu:~/Documents$ sudo fdisk -l
Disk /dev/sdb: 29.74 GiB, 31914983424 bytes, 62333952 sectors
Disk model: STORAGE DEVICE  
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x8ea7b95f

Device     Boot  Start     End Sectors  Size Id Type
/dev/sdb1  *     35296  100831   65536   32M  c W95 FAT32 (LBA)
/dev/sdb2       100832 2619391 2518560  1.2G  e W95 FAT16 (LBA)
/dev/sdb3            1    2079    2079    1M  e W95 FAT16 (LBA)
```


2.下载xfce img，使用dd命令将镜像烧录到TF卡
--
```
lazy@ubuntu:~/Documents$ bzcat openEuler-D1-xfce.img.bz2 | sudo dd of=/dev/sdb bs=1M iflag=fullblock oflag=direct conv=fsync status=progress
3242196992 bytes (3.2 GB, 3.0 GiB) copied, 180 s, 18.0 MB/s
3096+1 records in
3096+1 records out
3247012352 bytes (3.2 GB, 3.0 GiB) copied, 180.257 s, 18.0 MB/s
```

3.查看TF状态
--
```
lazy@ubuntu:~/Documents$ sudo fdisk -l /dev/sdb
Disk /dev/sdb: 29.74 GiB, 31914983424 bytes, 62333952 sectors
Disk model: STORAGE DEVICE  
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: EACEFD8D-68B3-493F-868A-5364E0A693D8

Device      Start     End Sectors  Size Type
/dev/sdb1   34784   35039     256  128K Linux filesystem
/dev/sdb2   35040   35295     256  128K Linux filesystem
/dev/sdb3   35296  100831   65536   32M Linux filesystem
/dev/sdb4  100832 6341820 6240989    3G Linux filesystem
```

4.对sdb4进行扩容
--
其中：
* first sector要和原来sdb4的start扇区保持一致
* last sector不用填写，直接回车就可以
* d：delete分区
* n:新建一个分区
* w：写入并退出

```
lazy@ubuntu:~/Documents$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

A hybrid GPT was detected. You have to sync the hybrid MBR manually (expert command 'M').

Command (m for help): d
Partition number (1-4, default 4): 4

Partition 4 has been deleted.

Command (m for help): n
Partition number (4-128, default 4): 4
First sector (100832-62333918, default 102400): 100832
Last sector, +/-sectors or +/-size{K,M,G,T,P} (100832-62333918, default 62333918): 

Created a new partition 4 of type 'Linux filesystem' and of size 29.7 GiB.
Partition #4 contains a ext4 signature.

Do you want to remove the signature? [Y]es/[N]o: n

Command (m for help): w
The device contains hybrid MBR -- writing GPT only. You have to sync the MBR manually.

The partition table has been altered.
Failed to update system information about partition 4: No such device or address

The kernel still uses the old partitions. The new table will be used at the next reboot. 
Syncing disks.

lazy@ubuntu:~/Documents$ 
```
5.调整分区大小，如果出现如下报错，需要拔插一下TF卡
--
```
lazy@ubuntu:~/Documents$ sudo resize2fs /dev/sdb4
resize2fs 1.45.5 (07-Jan-2020)
open: No such file or directory while opening /dev/sdb4
lazy@ubuntu:~/Documents$ sudo resize2fs /dev/sdb4
resize2fs 1.45.5 (07-Jan-2020)
Filesystem at /dev/sdb4 is mounted on /media/lazy/rootfs; on-line resizing required
old_desc_blocks = 12, new_desc_blocks = 119
The filesystem on /dev/sdb4 is now 31116540 (1k) blocks long.
```

至此镜像烧录完成

启动D1
==
1.串口线接入电脑
--
* Windows需要下载驱动（串口线驱动FT232R（串口线供应商提供版本）），macOS不需要
* windows需要到设备管理器中查询com口
* macOS需要在terminal里查看端口名称
```
ls /dev/cu.usbserial-*
```
* 波特率：115200
串口连上以后登录：root/openEuler12#$
使用命令w或uptime可以查看负载

2.HDMI连接D1
--
显示器可能不显示桌面，拔插一下HDMI，出现桌面
