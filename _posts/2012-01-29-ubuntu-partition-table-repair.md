---
layout: single
title: 为什么不应该贸然升级 ubuntu 和 gnome，以及如何修复分区表
excerpt: don't feed dogfood to your only compute
date: 2011-05-29 22:43
published: false
---

如果你只有一台电脑的话，就不要贸然升级，不论它看起来多美。

在我把 ubuntu 升级到 11.04 之后，系统就已经开始各种出错，不过还勉强能用。但是在 gnome3 发布的第一时间更新到 gnome3 则是个贸然的举动。启动后就只能停留在登录界面，不能选择用户，只有一个按钮，点开是重启、挂起、关机。<a href="http://forum.Ubuntu.org.cn/viewforum.php?f=49" target="_blank">很多人</a>都遇到了这个问题。据说杯具的源头是使用 ppa 源不能正确的安装 gnome-shell，这时可以进入恢复模式之后，使用 `apt-get install unity` 或者 `apt-get install gnome-shell` 来解决，但对我这个复杂的网络环境来讲，在命令行中连网也无比麻烦，于是这条路断了。

接下来的考虑是使用 livecd 来重新安装 Ubuntu，于是刻了个盘，启动，安装，一切正常——直到选择分区的时候——只能检测到硬盘，不能检测到分区信息。在<a href="http://forum.Ubuntu.org.cn/viewtopic.php?f=77&amp;t=194802" target="_blank">这里</a>（orz billbear）找到了相关的解决方案。原因就是分区表中有错误，扩展分区的结尾扇区超过了总扇区数。也就是parted爆的错误「Can't have a partition outside the disk!」

下面是我的分区表信息，重点关注红色部分

<pre>
ubuntu@ubuntu:~$ sudo fdisk -lu

Disk /dev/sda: 320.1 GB, 320072933376 bytes
255 heads, 63 sectors/track, 38913 cylinders, <span style="color: #ff0000;">total 625142448 sectors</span>
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x3dfce8ac

Device Boot Start End Blocks Id System
/dev/sda1 * 2048 20408319 10203136 7 HPFS/NTFS
/dev/sda2 20408320 20613119 102400 7 HPFS/NTFS（c盘）
/dev/sda3 20613120 174580399 76983640 7 HPFS/NTFS
/dev/sda4 174580434 <span style="color: #ff0000;">625153409</span> 225286488 f W95 Ext'd (LBA)
/dev/sda5 174583808 338423807 81920000 7 HPFS/NTFS（d盘）
/dev/sda6 338425353 368418640 14996644 83 Linux（挂载到/）
/dev/sda7 368418708 372418811 2000052 82 Linux swap / Solaris（swap）
/dev/sda8 372418893 382411252 4996180 83 Linux（挂载到/tmp）
/dev/sda9 382411323 625137338 121363008 83 Linux（挂载到/home）
</pre>

可见 sda4 的 end 比 total sectors 还要大。

我也不知道这个错误是怎么产生的，不过貌似大部分的电脑分区表都有这样或那样的错误，大部分情况下都不影响使用，但 linux 的 parted 是错误零容忍的，于是使用<a href="http://ubuntuforums.org/showthread.php?t=352723" target="_blank">这里</a>的方式手动修改分区表。然后又遇到了问题……

修改的分区表的思路就是先删除扩展分区，然后重新建立扩展分区和其中所有的分区，并且把错误的 end sector 修正过来，我顺利的删除了 sda4，然后重建了 sda4 和 sda5，但是当重建 sda6 的时候，明明在 start sector 输入了大于 sda5 end sector 的数值，依然提示「allocated」。

一番 google 无果，我最终放弃了解决这个问题，考虑到 11.04 之后已经很不稳定，而且一直想调整 /home 的分区大小无果，就决定直接删掉所有的 Ubuntu 分区重新安装。既然能正确的重建sda5，就能保住d盘，c盘(sda2)也不动，这样windows也没有影响，而且经测试重建了 sda4 就可以被 Ubuntu 安装程序正常识别分区信息，遂删除 sda6789。

重启之后发现系统还是 grub 引导的，然后 grub 找不到 ubuntu 的分区了。不过这个也在预想之中，进 livecd 重装 Ubuntu，问题解决，windows无恙。
