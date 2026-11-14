---
title: "RE"
date: 2026-07-19 10:15:49
categories:
  - CTF
  - 逆向
tags:
  - 逆向
  - RE
  - UPX
  - 脱壳
---

![](/img/re/re-01.png)

得到的ELF文件放进虚拟机里面会出现

放进ida里却没有相应的字符串，用DIE观察：

![](/img/re/re-02.png)

是upx壳，我用虚拟机里的upx脱壳之后得到脱壳后的re_unpacked

放进ida里面：

![](/img/re/re-03.png)

![](/img/re/re-04.png)

大概内容就是输入v11然后与a1进行对比所以a1就是我们想得到的flag。
