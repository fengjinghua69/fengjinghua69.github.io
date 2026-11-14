---
title: "CSre"
date: 2026-07-29 14:31:21
categories:
  - CTF
  - 逆向
tags:
  - 逆向
  - CSre
  - .NET
---

![](/img/csre/csre-01.png)

运行的时候会报错

放进ida里面看是一堆乱码，且搜索字符串也什么都没有

![](/img/csre/csre-02.png)

![](/img/csre/csre-03.png)

应该是加密了字符串？

![](/img/csre/csre-04.png)

![](/img/csre/csre-05.png)

经过搜索.NET要用dnSpy，放进去发现还是乱，应该是经过混淆，而且运行就会有异常，我想通过编辑来改掉，但是一改就有报错，改不掉，上网搜索之后知道可以用de4dot来改

![](/img/csre/csre-06.png)

![](/img/csre/csre-07.png)

这个smethod是用了SHA1，在里面解密得到flag
