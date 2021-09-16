- fdisk只能用于MBR分区，gdisk,parted可以用于GPT分区。
- fdisk大多数运维工作人员已经习惯这个交互模式。
- parted命令在创建删除分区使用命令比较方便，但是功能不是太完善，没有备份还原命令。
- gdisk在分区上命令和fdisk风格一样， 使用方便，学习难度低且功能强大，推荐使用。

---

- MBR：MBR分区表(即主引导记录)大家都很熟悉。所支持的最大卷：2T，而且对分区有限制：最多4个主分区或3个主分区加一个扩展分区
- GPT： GPT（即GUID分区表）。是源自EFI标准的一种较新的磁盘分区表结构的标准，是未来磁盘分区的主要形式。与MBR分区方式相比，具有如下优点。突破MBR 4个主分区限制，每个磁盘最多支持128个分区。支持大于2T的分区，最大卷可达18EB。

---

parted命令常用选项。
当在命令行输入parted后，进入parted命令的交互模式。输入help会显示帮助信息。下面就简单介绍一下常用的功能

```
1、Check     简单检查文件系统。建议用其他命令检查文件系统，比如fsck
2、Help      显示帮助信息
3、mklabel   创建分区表， 即是使用msdos（MBR）还是使用gpt，或者是其他方式分区表
4、mkfs      创建文件系统。该命令不支持ext3 格式，因此建议不使用，最好是用parted分好区，然后退出parted交互模式，用其他命令进行分区，比如：mkfs.ext3
5、mkpart    创建新分区。
        格式：mkpart PART-TYPE  [FS-TYPE]  START  END
             PART-TYPE 类型主要有primary（主分区）, extended（扩展分区）, logical（逻辑区）. 扩展分区和逻辑分区只对msdos。
             fs-type   文件系统类型，主要有fs32，NTFS，ext2，ext3等
             start end 分区的起始和结束位置。
6、mkpartfs  建立分区及其文件系统。目前还不支持ext3文件系统，因此不建议使用该功能。最后是分好区后，退出parted，然后用其他命令建立文件系统。
7、print    输出分区信息。该功能有3个选项，
       free 显示该盘的所有信息，并显示磁盘剩余空间
     number 显示指定的分区的信息
        all 显示所有磁盘信息
8、resize   调整指定的分区的大小。目前对ext3格式支持不是很好，所以不建议使用该功能。
9、rescue   恢复不小心删除的分区。如果不小心用parted的rm命令删除了一个分区，那么可以通过rescue功能进行恢复。恢复时需要给出分区的起始和结束的位置。然后parted就会在给定的范围内去寻找，并提示恢复分区。
10、rm      删除分区。命令格式 rm  number 。如：rm 3 就是将编号为3的分区删除
11、select  选择设备。当输入parted命令后直接回车进入交互模式是，如果有多块硬盘，需要用select 选择要操作的硬盘。如：select /dev/sdb
12、set     设置标记。更改指定分区编号的标志。标志通常有如下几种：boot  hidden   raid   lvm 等。boot 为引导分区，hidden 为隐藏分区，raid 软raid，lvm 为逻辑分区。如：set 3  boot  on   设置分区号3 为启动分区
```
