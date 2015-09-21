---
layout: post
title: "Archlinux Windows10 UEFI安装教程"
subtitle: "preinstalled Windows10 dual boot"
date: 2016-10-14 17:10:00 +0800
author: "Roach"
catalog: true
tags:
    - Archlinux
---

> 有些事情不试过之后，怎么甘心。

## 准备

首先，需要刻录一个Arch安装U盘。

我的当前电脑已经安装了Windows10,并且是`GPT`分区格式`UEFI`引导。

可以从Arch Linux的[官方网站](https://www.archlinux.org/)下载用于安装Arch Linux的iso镜像。而为了加速，世界上有很多开源社区建立了开源镜像站，里面提供很多发行版的iso镜像及软件源。下面我们从中科大的开源镜像站([http://mirrors.ustc.edu.cn/](http://mirrors.ustc.edu.cn/))下载Arch Linux的iso.右侧有个“获取安装镜像”点击选择即可，下载2016.10.01版本的镜像，文件名为archlinux-2016.10.01-dual.iso.

下载过iso之后可以刻录光盘，也可以刻录U盘。Windows刻录U盘可以使用[Rufus](https://rufus.akeo.ie/)制作启动盘。

首先下载Rufus.exe，打开就能看到主界面。插入U盘之后，可以看到Rufus可以识别出U盘。Rufus有很多设置项，其中有一项叫“新卷标”。可以通过windows问家管理器打开镜像，镜像的名字叫做`ARCH_201610`.接着，我们在Rufus的“新卷标”处填写`ARCH_201610`，分区方案选择“用于BIOS或MBR计算机的UEFI分区方案”。然后在“创建一个启动盘使用”那里，右边有个光盘的图标，点击并选择下载下来的iso文件。最后点“开始”制作启动U盘。

接着Rufus会提示下载syslinux，选择“是”。

然后提示选择写盘模式，我们使用推荐的ISO模式。

最后有个格式化的警告，点击确定，即可格式化U盘并制作启动盘。

然后，就可插入安装盘启动了。推荐使用网线安装arch，这样进入安装系统就已经可以上网进行安装。而不用进行`Wifi`的配置。

---

## 分区&格式化

**分区**

首先，我们需要对硬盘进行分区操作。GPT分区就可以使用gdisk来进行分区操作。

```
# gdisk /dev/sda
GPT fdisk (gdisk) version 1.0.1

The protective MBR's 0xEE partition is oversized! Auto-repairing.

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help):
```

这时候，你输入`？`，就可以告诉你各种操作的命令，比如`p`输出你当前硬盘的分区信息、`n`是新建一个分区、`w`该改动写入分区表（数据重要，谨慎处理！）、`q`退出。

```
Command (? for help): p
Disk /dev/sda: 1465149168 sectors, 698.6 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): 324BC003-A13E-470C-9117-F4B42789BA44
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 1465149134
Partitions will be aligned on 2048-sector boundaries
Total free space is 542404269 sectors (258.6 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
1            2048          923647   450.0 MiB   2700  Basic data partition
2          923648         1128447   100.0 MiB   EF00  EFI system partition
3         1128448         1161215   16.0 MiB    0C01  Microsoft reserved ...
4         1161216       230688767   109.4 GiB   0700  Basic data partition
5       230688768       398458879   80.0 GiB    0700  Basic data partition
6       398458880       639631359   115.0 GiB   0700  Basic data partition
7       639631360       901775359   125.0 GiB   0700  Basic data partition
8       901775360       922746879   10.0 GiB    0700  Basic data partition
```

UEFI模式，Windows下已经有一个EFI分区，这个分区我们也可以使用，就不用创建EFI分区了（Linux下你想创建一个也可以）。而这样我们就需要将EFI（sda2）挂载在，/boot的efi目录下。

先创建一个boot分区：500M。

```
Command (? for help): n
Partition number (9-128, default 9):
First sector (34-1465149134, default = 922746880) or {+-}size{KMGTP}:
Last sector (922746880-1465149134, default = 1465149134) or {+-}size{KMGTP}: +500M
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'

Command (? for help): p

data partition
7       639631360       901775359   125.0 GiB   0700  Basic data partition
8       901775360       922746879   10.0 GiB    0700  Basic data partition
9       922746880       923770879   500.0 MiB   8300  Linux filesystem
```

+ 因为之前已经有了8个分区了，所以新建的默认是sda9,我们直接Enter就好。
+ 然后下一行告诉你新建分区的首个分区位置，不修改直接Enter。
+ 再一行是新建分区的结束扇区（默认剩余所有空间），+500M表示我们只用500M。
+ 然后再下面两行让你确定你新建分区的格式，Enter表示8300,即linux分区格式。

这样我们就创建好了一个空间500M的分区。然后，我又创建了一个4G的swap分区、70G的根分区、120G的home分区。**需要注意的是，创建swap分区时将8300 -> 8200**。

```
Number  Start (sector)    End (sector)  Size       Code  Name
1            2048          923647   450.0 MiB   2700  Basic data partition
2          923648         1128447   100.0 MiB   EF00  EFI system partition
3         1128448         1161215   16.0 MiB    0C01  Microsoft reserved ...
4         1161216       230688767   109.4 GiB   0700  Basic data partition
5       230688768       398458879   80.0 GiB    0700  Basic data partition
6       398458880       639631359   115.0 GiB   0700  Basic data partition
7       639631360       901775359   125.0 GiB   0700  Basic data partition
8       901775360       922746879   10.0 GiB    0700  Basic data partition
9       922746880       923770879   500.0 MiB   8300  Linux filesystem
10       923770880       932159487   4.0 GiB     8200  Linux swap
11       932159488      1078960127   70.0 GiB    8300  Linux filesystem
12      1078960128      1330618367   120.0 GiB   8300  Linux filesystem
```

**格式化**

除了swap分区，将另外的分区格式化为ext4格式

```
mkfs -t ext4 /dev/sda9
mkfs -t ext4 /dev/sda11
mkfs -t ext4 /dev/sda12

mkswap /dev/sda10
```

---

## 安装

安装之前，我们先文件系统（你要安装的系统）挂在到镜像U盘中的/mnt目录下，而EFI分区挂载到/boot/efi目录下

```
mount /dev/sda11 /mnt
接着挂载/boot和/home,注意要先建立目录才能挂载。

install -d /mnt/{boot,home}
mount /dev/sda9 /mnt/boot
mkdir /mnt/boot/efi
mount /dev/sda2 /mnt/boot/efi
mount /dev/sda12 /mnt/home
swapon /dev/sda10
```

我们还需要修改一下pacman源，改为国内的源可以提升安装速度：`vi /etc/pacman.d/mirrorlist`之保留`China`大陆的源。

**安装基本系统**

```
$ pacstrap /mnt base base-devel
```

**生成/etc/fstab文件**

它的作用是指示init程序挂载分区到指定的挂载点。

```
$ genfstab -U -p /mnt >> /mnt/etc/fstab
```

**Chroot到安装在硬盘中的系统**

我们已经知道，文件系统是一个树形结构，它有一个根目录/,每个文件路径都可以从/一步一步走下去找到。而chroot的功能就是把一个进程(及其子进程)的根目录改为指定的路径，这样这个进程找一个文件时，便会从chroot指定的目录开始查找。

在这里，我们chroot到已经安装好的那个文件系统里面，这样就很容易对我们新安装的系统进行操作了。

一般来说，为了在进入新的文件系统后仍可以方便地操作内核，要挂载/proc,/dev,/sys到新的文件系统中。Arch的安装工具中提供了一个叫arch-chroot的脚本，它封装了这些操作，简化了chroot的过程。

现在我们执行arch-chroot /mnt，这样就以chroot的方式进入了新的系统。

```
$ arch-chroot /mnt /bin/bash
```

---

## 配置系统

**Root password**

```
$ passwd
```


**Hostname**

```
$ echo arch >> /etc/hostname
$ vi /etc/hosts
```

/etc/hosts should look like:

```
127.0.0.1   localhost.localdomain   arch
::1         localhost.localdomain   arch
```

**Timezone**

```
$ ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ hwclock --systohc --utc
```

**Locale**

```
$ vi /etc/locale.gen
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
$ locale-gen
$ echo LANG=en_US.UTF-8 > /etc/locale.conf
```

**Initial ramdisk environment**

可能会有一些提示信息，安装过显卡驱动就可一了。

```
$ mkinitcpio -p linux
```

**Grub**

```
$ pacman -S os-prober grub efibootmgr
$ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub --recheck
$ grub-mkconfig -o boot/grub/grub.cfg
$ sudo update-grub
```

但是，`os-prober`输出信息显示，并没有自动判别出来windows，需要我们自己修改`/boot/grub/grub.cfg`。如果显示添加了windows引导项，下面那一步就免了。

```
menuentry "Microsoft Windows10" {
    insmod part_gpt
    insmod fat
    insmod search_fs_uuid
    insmod chain
    search --fs-uuid --set=root $hints_string $fs_uuid
    chainloader /EFI/Microsoft/Boot/bootmgfw.efi
}
```

Now change:

- `$hints_string` by the output of `$ grub-probe --target=fs_uuid /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi`
- `$fs_uuid` by the output of `$ grub-probe --target=hints_string /boot/efi/EFI/Microsoft/Boot/bootmgfw.efi`

*注意*：如果这里不添加Windows10启动项，安装Gnome桌面之后重新生成`grub.cfg`会自动生成启动项。

**网络连接管理器**

```
$ pacman -S iw wpa_supplicant networkmanager
```

如果退出chroot之后，进入arch，使用Wifi，那么就可以禁用dhcp，让networkmanager自启。如果是有线就暂时不进行下面的操作。使用networkmanager的原因是和Gnome3结合的很好，并且可以管理有线和wifi。

```
# Wifi 执行
$ systemctl disable dhcpcd
$ systemctl enable NetworkManager
```

**退出Chroot**

```
$ exit
$ umount -R /mnt
$ reboot
```

这样就进入arch了！

---

## 配置Arch

**联网**

有限则已经联网，或者`ip link`看一下你的有限网卡的名字Name，使用`dhcpcd Name`联网

Wifi：

```
$ nmcli nm wifi on
$ nmcli dev wifi connect <network ssid> password <password>
```

**安装zsh**

```
$ pacman -S zsh
```


**创建用户**

```
$ useradd -m -g users -G wheel -s /bin/zsh roach
$ passwd roach
$ chfn roach
```

最后设置wheel组的用户能用sudo获取root权限，使用visudo来修改sudo的配置：

`$ visudo`

找到这样的一行,把前面的#去掉，然后按ESC键，输入:x!回车就可以保存并退出：

`#%wheel ALL=(ALL) ALL`

**添加archlinuxcn源**

archlinuxcn是一个由arch中文社区维护的镜像源，其中包含了许多官方镜像中没有但又经常使用的软件。可以通过编辑pacman.conf文件来添加archlinuxcn源:

`sudo vi /etc/pacman.conf`
在文档结尾处加入下面的文字：

```
[archlinuxcn]
SigLevel = Optional TrustAll
Server = http://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

完成之后刷新pacman数据库`sudo pacman -Syy`

安装archlinuxcn-keyring，这个是提供校验软件包的密钥的。`sudo pacman -S archlinuxcn-keyring`现在archlinuxcn源就可以使用了。

安装`yaourt`：`sudo pacman -S yaourt`

yaourt相当于一个加强版的pacman，在pacman的基础上添加了对AUR的支持，并提供诸如彩色输出、交互式搜索模式等一系列实用功能。yaourt包含在archlinuxcn源中.

**Install libselinux-2.5-3**

在安装gnome之前安装，不然会报`GLib-GIO-Message: Using the 'memory' GSettings backend`错误。

**Install X**

触摸板: `$ pacman -S xf86-input-synaptics`

```
$ pacman -S xorg-server xorg-server-utils xorg-xinit
```

让选择的选项直接Enter（默认，没毛病。。。

**显卡驱动**

```
$ pacman -S mesa xf86-video-intel nvidia bbswitch bumblebee nvidia-utils
$ gpasswd -a roach bumblebee
$ systemctl enable bumblebeed
```

**安装Gnome**

```
$ pacman -S gnome network-manager-applet
$ systemctl enable gdm.service

# NetworkManage
$ systemctl disable dhcpcd
$ systemctl enable NetworkManager
```

**中文输入法**

```
$ pacman -S fcitx-im fcitx-configtool fcitx-fbterm kcm-fcitx
$ pacman -S fcitx-googlepinyin fcitx-cloudpinyin
```

之所以使用谷歌拼音，是搜狗的并不符合fcitx规范，和Gnome Input Method扩展不兼容。

打开~/.xprofile文件并加入以下几行：

```
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```

然后在fcitx配置中的`Input Method`选项添加`Keyboard-Chinese`和`Google Pinyin`，第一项配置下方说是不激活的，但是这样也可以正常工作。

**美化zsh**

```
# pacman -S zsh git
# chsh -s $(which zsh)

$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

---

## 后续工作

#### 扩展

1. Drop Down Terminal    #guake 在Gnome3.20工作不好
2. Applications Menu
3. Input Method Panel
4. Places Status Indicator

#### pacman

```
pacman -Syy 在安装软件前我们需要先和软件源进行一次同步

pacman -Syu 同步之后可以选择批量更新

pacman -S package 在源里搜索并安装单个软件包，会自动安装其依赖

pacman -U /path/to/package/package_name.pkg.tgz安装从其他地方下载到本地的软件包

pacman -Ss package安装某个软件前可以先在源里搜索一有没有

pacman -Qs package类似的在本地搜索软件包

pacman -Rs package删除某个软件包，以及只有其用到的依赖包，如果要保留依赖可以只指定-R

pacman -Qdt 要罗列所有不再作为依赖的软件包(孤立orphans)

pacman -Qet 要罗列所有明确安装而且不被其它包依赖的软件包

pacman -R $(pacman -Qdtq) autoremove
```

#### atom

`yaourt -S atom-editor-bin` AUR，需要非root用户安装。

#### 双系统时间设置

之所以 Windows 与 Linux 双系统之间有时间差，是因为这两个系统使用了不同的方式来识别硬件时钟（Hardware Clock）。Linux 将硬件时钟当作 UTC 时间，而 Windows 将硬件时钟当作本地时间（ Local time）。由于时间的处理方式不同，Windows 不管重启多少次都识别 Local time，时间都不会改变。而当我们从 Linux 重启到 Windows 时，硬件时钟已经被 Linux 认为 UTC 方式，而 Windows 再将其强制转换成 Local time，这就造成了时间差。

解决方法： 修改Windows注册表

> To fix it, just hit Start and type regedit.exe in the search box. Hit Enter and navigate to HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation. Right click anywhere in the right pane and hit New > DWORD (64-bit) Value. Name it RealTimeIsUniversal, then double click on it and give it a value of 1.

---

## grub rescue
如果，你在windows系统中修改分区表了（创建、删除分区等），那么一般情况下再次重启系统是grub是无法成功完成信道的会进入一个命令行：

```
grub rescue
grub>
```

下面的方法，只测试了`grub`而非`grub legacy`。grub也就是grub2.

这时候，如果你想修复一下引导，有两种方法：

#### 方法一

这时候，你可你输入`ls`命令会列出在grub命令模式下，分区的名字。你总应该记得自己的grub引导是装在那个分区的吧（假设装载/dev/sda9，并且挂在在/boot目录下）。

```
grub> linux (hd0,gpt9)/vmlinuz root=/dev/sda3 noresume 3
grub> initrd (hd0,gpt9)/initrd
grub> boot
```

上面需要注意的是，在输入`vmlinu`后按下`tab`键，应该可以自动补全，同样，`initrd`之后也是。这样，在你输入`reboot`之后，就可以进入系统了。然后重新生成`grub.cfg`就可以了。

#### 方法二

经历过arch的安装以及arch-chroot，最直接的想法就是，我们通过安装盘挂载分区到安装时正确的目录。在通过chroot进入后，重新生成`grub.cfg`。就好了，这种方法也可以在重装windows后，再通过`grub-install`重新安装grub，恢复引导。

---

## References

1. [Installation guide (简体中文)](https://wiki.archlinux.org/index.php/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#.E5.BB.BA.E7.AB.8B.E7.A1.AC.E7.9B.98.E5.88.86.E5.8C.BA)
2. [安装Arch Linux：一步一步了解Linux系统](https://github.com/mytbk/Linux_Notes/blob/master/install/install-archlinux.md)
3. [ArchLinux安装与配置小结](https://pannzh.github.io/tech/2015/12/06/arch-linux-note.html)
4. [Arch Linux installation (preinstalled Windows 8.1 dual boot)](https://gist.github.com/miguelfrde/5dde43aa08b076106b9e)
5. [入教教程 —— 安装arch](http://wawei.coding.me/2015/12/09/Enjoy-arch-1/)
6. [配置和美化Arch Linux](http://www.jianshu.com/p/fe2165cc6af8)
7. [Pacman (简体中文)](https://wiki.archlinux.org/index.php/Pacman_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#.E5.88.A0.E9.99.A4.E8.BD.AF.E4.BB.B6.E5.8C.85)
8. [如何解决Ubuntu与Windows双系统时间不同步](https://www.sysgeek.cn/fix-time-difference-between-ubuntu-windows/)
9. [Fix Incorrect Clock Settings in Windows When Dual-Booting with OS X or Linux](http://lifehacker.com/5742148/fix-windows-clock-issues-when-dual-booting-with-os-x)
10. [[opensuse] boot without Grub's menu](https://lists.opensuse.org/opensuse/2012-11/msg00014.html)
