---
layout: post
title: "Linux command learning"
subtitle: "step by step, bit by bit"
date: 2015-12-06 19:24:26 +0800
author: "Roach"
catalog: true
tags:
    - Linux
---

> To love it!

### fg

当用 *Ctrl+Z* 挂起一个程序，这时你可以使用 **fg** 让它重新在前台运行，也可以使用 **bg** 让它在后台运行；

**jobs** 命令可以看当前有那些后台任务，每个列出的任务前面有命令编号；可以根据编号结合 **fg** 选择将哪个任务重新放回前台；

当需要关闭一个后台进程时，可以 *kill it by pid*，同样可以使用命令 **kill %num**，其中 *num* 是 **jobs** 命令所列后台任务的编号。

---

### localectl

localectl: [Control the system locale and keyboard layout settings](https://docs.fedoraproject.org/en-US/Fedora/23/html/System_Administrators_Guide/ch-System_Locale_and_Keyboard_Configuration.html)

```
$ localectl status
   System Locale: LANG=zh_CN.UTF-8
       VC Keymap: cn
      X11 Layout: cn
```

---

### netstat

*netstat -anp  \| grep 8787* : 查看 *8787* 端口被哪个程序占用；

### systemd-analyze

systemd-analyze - Analyze system boot-up performance

usage: systemd-analyze [blame] 

[linux 开机速度慢——Why does plymouth-quit-wait.service take ~20 s to finish?](https://www.reddit.com/r/linuxquestions/comments/3qfmjk/why_does_plymouthquitwaitservice_take_20_s_to/)

[Journal size limit](https://wiki.archlinux.org/index.php/Systemd#Journal_size_limit)