---
title: "check_1n"
date: 2026-08-03 10:26:27
categories:
  - CTF
  - 逆向
tags:
  - 逆向
  - check_1n
  - Base64
---

运行之后有如下提示

![](/img/check-1n/check-1n-01.png)

摁下空格之后出现如下界面。摁下上下左右键即可操控图示键盘

![](/img/check-1n/check-1n-02.png)

![](/img/check-1n/check-1n-03.png)

无壳，放进ida搜字符串

![](/img/check-1n/check-1n-04.png)

![](/img/check-1n/check-1n-05.png)

发现有密码错误的提示，跟进去发现密码是hello world

![](/img/check-1n/check-1n-06.png)

![](/img/check-1n/check-1n-07.png)

Base64解密让我们去玩打砖块，玩完就是flag
