# [centos7] use SSM to manager lvm

Logical Volume Manager (LVM) is an extremely flexible disk management scheme, allowing you to create and resize logical disk volumes off of multiple physical hard drives with no downtime. However, its powerful features come with the price of somewhat steep learning curves, with more involved steps to set up LVM using multiple command line tools, compared to managing traditional disk partitions.

逻辑卷管理器（LVM）是一种非常灵活的磁盘管理方案，允许您在不停机的情况下从多个物理硬盘驱动器创建逻辑磁盘卷并调整其大小。 然而，其强大的功能与价格有点陡峭的学习曲线，与管理传统磁盘分区相比，更多涉及到使用多个命令行工具设置LVM的步骤。

这对于CentOS / RHEL用户来说是个好消息。 最新的CentOS / RHEL 7现在配备了System Storage Manager（又称ssm），它是Red Hat开发的统一命令行界面，用于管理各种存储设备。 目前有三种可用于ssm的卷管理后端：LVM，Btrfs和Crypt。

Here is good news for CentOS/RHEL users. The latest CentOS/RHEL 7 now comes with System Storage Manager (aka ssm) which is a unified command line interface developed by Red Hat for managing all kinds of storage devices. Currently there are three kinds of volume management backends available for ssm: LVM, Btrfs, and Crypt.

## 安装

```
sudo yum install -y system-storage-manager
```

## 列表查看

```
ssm list

[root@dockers dockers]# ssm list
----------------------------------------------------------------
Device        Free      Used      Total  Pool        Mount point
----------------------------------------------------------------
/dev/fd0                        4.00 KB                         
/dev/sda                      100.00 GB              PARTITIONED
/dev/sda1                       1.00 GB              /boot      
/dev/sda2  4.00 MB  98.99 GB   99.00 GB  cl_dockers             
/dev/sdb                      100.00 GB                         
/dev/sdc                      100.00 GB                         
----------------------------------------------------------------
------------------------------------------------------
Pool        Type  Devices     Free      Used     Total  
------------------------------------------------------
cl_dockers  lvm   1        4.00 MB  98.99 GB  99.00 GB  
------------------------------------------------------
----------------------------------------------------------------------------------------------
Volume                Pool        Volume size  FS      FS size       Free  Type    Mount point
----------------------------------------------------------------------------------------------
/dev/cl_dockers/root  cl_dockers     50.00 GB  xfs    49.98 GB   48.52 GB  linear  /          
/dev/cl_dockers/swap  cl_dockers      7.88 GB                              linear             
/dev/cl_dockers/home  cl_dockers     41.12 GB  xfs    41.10 GB   41.07 GB  linear  /home      
/dev/sda1                             1.00 GB  xfs  1014.00 MB  846.48 MB  part    /boot      
----------------------------------------------------------------------------------------------
```

## 创建LVM池以及LVM卷

```
1. ssm create --help
2. ssm create -p lvm_docker_pool -n docker_vol -s 100G --fstype ext4 /dev/sdb /dev/sdc

[root@dockers dockers]# ssm create -p lvm_docker_pool -n docker_vol -s 100G --fstype ext4 /dev/sdb /dev/sdc
  Volume group "lvm_docker_pool" successfully created
WARNING: ext4 signature detected on /dev/lvm_docker_pool/docker_vol at offset 1080. Wipe it? [y/n]: y
  Wiping ext4 signature on /dev/lvm_docker_pool/docker_vol.
  Logical volume "docker_vol" created.
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
6553600 inodes, 26214400 blocks
1310720 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2174746624
800 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

## 变更卷大小

```
1. ssm resize --help
2. ssm resize -s +20G /dev/lvm_docker_pool/docker_vol

[root@dockers dockers]# ssm resize -s +20G /dev/lvm_docker_pool/docker_vol 
fsck from util-linux 2.23.2
/dev/mapper/lvm_docker_pool-docker_vol: clean, 11/6553600 files, 459544/26214400 blocks
  Size of logical volume lvm_docker_pool/docker_vol changed from 100.00 GiB (25600 extents) to 120.00 GiB (30720 extents).
  Logical volume lvm_docker_pool/docker_vol successfully resized.
resize2fs 1.42.9 (28-Dec-2013)
Resizing the filesystem on /dev/mapper/lvm_docker_pool-docker_vol to 31457280 (4k) blocks.
The filesystem on /dev/mapper/lvm_docker_pool-docker_vol is now 31457280 blocks long.
```

## 移除卷或者池

```
ssm remove /dev/lvm_docker_pool/docker_vol
ssm remove lvm_docker_pool
```

## 快照管理

```
ssm snapshot

[root@dockers dockers]# ssm snapshot --help
usage: ssm snapshot [-h] [-s SIZE] [-d DEST | -n NAME] volume

positional arguments:
  volume                Volume, or mount point to take a snapshot of.

optional arguments:
  -h, --help            show this help message and exit
  -s SIZE, --size SIZE  Gives the size to allocate for the new snapshot
                        volume. A size suffix K|k, M|m, G|g, T|t, P|p, E|e can
                        be used to define 'power of two' units. If no unit is
                        provided, it defaults to kilobytes. This is optional
                        and if not given, the size will be determined
                        automatically. Additionally the new size can be
                        specified as a percentage of the original volume size
                        (50%), as a percentage of free pool space (50%FREE),
                        or as a percentage of used pool space (50%USED).
  -d DEST, --dest DEST  Destination of the snapshot specified with absolute
                        path to be used for the new snapshot. This is optional
                        and if not specified default backend policy will be
                        performed.
  -n NAME, --name NAME  Name of the new snapshot. This is optional and if not
                        specified default backend policy will be performed.
```
