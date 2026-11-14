---
title: "[ACTF新生赛2020]rome"
date: 2026-05-17 19:16:31
categories:
  - CTF
  - 逆向
tags:
  - 逆向
  - ACTF新生赛2020
  - rome
---

![](/img/rome/rome-01.png)

![](/img/rome/rome-02.png)

搜索字符串发现了正确的提示，点进去查看代码

输入内容后经历判断，点击后面的v6v7之类的变量，发现他们就是连续的我们输入的内容

![](/img/rome/rome-03.png)

上面的判断是确定外壳是ACTF{}

![](/img/rome/rome-04.png)

之后把内里的内容赋给v1，判断大小值进行变换

![](/img/rome/rome-05.png)

之后再把新的v1和text进行判断

所以把text倒回去就可以得到flag

![](/img/rome/rome-06.png)
