---
title: "java逆向"
date: 2026-05-17 18:34:27
categories:
  - CTF
  - 逆向
tags:
  - 逆向
  - java逆向
---

![](/img/java-ni-xiang/java-ni-xiang-01.png)

题目是要求输入字符串经历变换之后与key进行比较，

变换如下：（c+“@”）^32，然后整个流程就是（c+64）^32=key[i]

而要得到c，就是(key[i]^32)-64

![](/img/java-ni-xiang/java-ni-xiang-02.png)
