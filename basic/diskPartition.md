# 设备在linux中的档名

​	

| 装置                | 装置档名                                                     |
| ------------------- | ------------------------------------------------------------ |
| SCSI/SATA/USB硬碟機 | /dev/sd[a-p]                                                 |
| U盘                 | /dev/sd[a-p]                                                 |
| VirtI/O界面         | /dev/vd[a-p]\(用于虚拟机内)                                  |
| 软碟机              | /dev/fd[0-1]\(Floopy Disk)                                   |
| 打印机/印表機       | /dev/lp[0-15]                                                |
| 鼠标                | /dev/input/mouse[0-15]; /dev/mouse(当前鼠标); /dev/psaux (PS/2界面) |
| CDROM/DVDROM        | /dev/scd[0-1]; /dev/sr[0-1]; /dev/cdrom(当前)                |
| 磁带机              | /dev/tape(当前); /dev/st0(sata接口); /dev/ht0(ide接口)       |

[更多设备档名](https://www.kernel.org/doc/html/v4.20/admin-guide/devices.html)

***

# 磁盘分割

磁盘可分为磁区(Sector)与磁轨(track)两种单位，磁区的物理量设计有两种，分别是512bytes和4Kbytes.

![](images/Disktructure.jpg)

## MBR与GPT分割表

1. MBR分割表与限制

开机管理程序记录区与分割表放在磁盘第一个磁区，通常为512bytes。所以，第一个磁区有：

* 主要开机记录区：安装开机管理程序的地方，有446bytes
* 分割表(partititon table): 记录硬盘分割的状态，有64bytes

由于只有64bytes，因此最多能有四组记录区，每组记录区记录了该区段起始与结束的磁柱号码。

![分割表示例图](images/partition-1.png)



假如上述装置名为/dev/sda,那么分区依次为:

* p1 = /dev/sda1
* p2 = /dev/sda2
* p3 = /dev/sda3
* p4 = /dev/sda4

这四个分区也被称为主要分区(primary)或是延伸(extended)分割槽

分割槽最小单位通常为磁柱(cylinder)

<img src="images/partition-2.png"  />

利用额外的磁区记录分割信息，p1是主要分割，p2是延伸分割，切割出来的五个部分称为逻辑分区

逻辑分区档名：

* L1: /dev/sda5
* L2: /dev/sda6
* L5: /dev/sda9

每组分割表仅有16bytes，所以能记录的地址是有限的，无法支持2.2T以上的容量。

mbr仅有一块区块，一旦破坏，很难救援

主要分割和延伸分割最多有四个(硬盘限制)

延伸分割最多只能有一个(系统限制)。延伸分区无法格式化



## GUID partition table, GPT磁盘分割表

GPT用逻辑区块地址(Logical Block Address)来记录。

![](images/gpt_partition_1.jpg)

LBA 0 用来兼容mbr

LBA 1 记录分割位置以及备份位置

LBA 2-33  每个LBA都可记录4个分割记录

## bios与UEFI开机检测程序

### bios配mbr/gpt

开机流程：

1. BIOS
2. MBR
3. 开机管理程序(boot loader)
4. 核心档案(开始系统)

boot loader只是系统安装在mbr上的一套程序罢了

boot loader有多种功能：

* 提供选单：选择不同开机项目
* 载入核心档案
* 转交其他boot loader: 将开机管理程序交给其他loader负责

![](images/loader.gif)

### uefi bios配gpt

| 比较项目       | 传统bios                                               | uefi         |
| -------------- | ------------------------------------------------------ | ------------ |
| 使用语言       | 组合语言                                               | c语言        |
| 硬件资源控制   | 使用中断(IRQ)管理；不可变的记忆体存储；不可变输入/输出 | 使用驱动     |
| 运作环境       | 16位                                                   | cpu保护模式  |
| 扩充方式       | 16位                                                   | 直接载入驱动 |
| 第三方厂商支持 | 较差                                                   | 较好         |
| 图形化         | 较差                                                   | 较好         |
|                |                                                        |              |

# 磁盘分割

## 目录树(directory tree)

![](images/dirtree.gif)

## 档案系统与目录树关系

![](images/dir_3.png)

## 分区规划

初次接触linux：只要分割/和swap即可

