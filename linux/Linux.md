## Linux
- 硬盘扩容
```
# 创建新分区，挂载
fdisk /dev/sda

Command (m for help): m
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)

Command (m for help): n
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
Select (default p): p
Partition number (3,4, default 3): 3
First sector (41943040-62914559, default 41943040): 
Using default value 41943040
Last sector, +sectors or +size{K,M,G} (41943040-62914559, default 62914559): 
Using default value 62914559
Partition 3 of type Linux and of size 10 GiB is set

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.

# 重启系统
reboot

# 创建物理卷
pvcreate /dev/sda3

# 查看物理卷信息
pvdisplay

# 新增加的分区/dev/sda3加入到根目录分区
vgextend centos /dev/sda3

# 查看卷组信息
vgdisplay

# 增加centos大小
lvresize -L +10G /dev/mapper/centos-root

# 报错提示：一个PE的大小为4M，所以总大小为 2559 × 4M
[root@docker ~]# lvresize -L +10G /dev/mapper/centos-root
  Insufficient free space: 2560 extents needed, but only 2559 available
[root@docker ~]# lvresize -L +10236M /dev/mapper/centos-root
  Size of logical volume centos/root changed from <17.00 GiB (4351 extents) to 26.99 GiB (6910 extents).
  Logical volume centos/root successfully resized.

# 重新识别centos大小
xfs_growfs /dev/mapper/centos-root

# 查看扩容后的大小
df -h
```