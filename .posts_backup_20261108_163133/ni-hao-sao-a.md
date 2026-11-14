---
title: "[NPUCTF2020]你好sao啊"
date: 2026-08-06 20:02:05
categories:
  - CTF
  - 逆向
tags:
  - 逆向
  - NPUCTF2020
  - 你好sao啊
  - Base64
---

![](/img/ni-hao-sao-a/ni-hao-sao-a-01.png)

![](/img/ni-hao-sao-a/ni-hao-sao-a-02.png)

![](/img/ni-hao-sao-a/ni-hao-sao-a-03.png)

Ida里面就能搜到我们看见的字符串

![](/img/ni-hao-sao-a/ni-hao-sao-a-04.png)

s是输出，把s送到RxEncode函数做变换，得到s1，s1=s2且s长度为32就是对的，我们有s2，要把s2倒回去，得到flag

![](/img/ni-hao-sao-a/ni-hao-sao-a-05.png)

![](/img/ni-hao-sao-a/ni-hao-sao-a-06.png)

这是个base64解码，表变了，s2=9E 9B 9C B5 FE 70 D3 0F B2 D1 4F 9C 02 7F AB DE 59 65 63 E7 40 9D CD FA 04

![](/img/ni-hao-sao-a/ni-hao-sao-a-07.png)
