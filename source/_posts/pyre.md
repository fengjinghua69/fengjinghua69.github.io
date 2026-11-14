---
title: "[GWCTF 2019]pyre"
date: 2026-05-17 18:34:27
categories:
  - CTF
  - 逆向
tags:
  - 逆向
  - GWCTF 2019
  - pyre
  - 异或
---

用pycdc反编译得到flag

![](/img/pyre/pyre-01.png)

我们输入内容之后，累加再把他们进行异或

![](/img/pyre/pyre-02.png)

末尾值直接还原，然后从后往前异或

```
a[l-1] = b[l-1]
for i from l-2 down to 0:
    a[i] = b[i] ^ a[i+1]
```

得到flag
