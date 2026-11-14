---
title: "findit"
date: 2026-05-17 18:34:27
categories:
  - CTF
  - 逆向
tags:
  - 逆向
  - findit
---

解压得到一个apk，放到jadx，发现给出了两个字符串，a之后分情况赋值给x，转换成string类型给m，同样的b之后分情况赋给了y然后转换类型给了n

![](/img/findit/findit-01.png)

中间把输入与m进行对比之后，若正确则返回n，只要按照要求执行一遍得到n即可

得到flag
