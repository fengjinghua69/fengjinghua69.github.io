---
title: the usage of DOSBOX
date: 2026-11-07 21:13:53
categories:
  - 笔记
tags:
  - DOSBOX
  - 工具
---
我们可以使用DOSBOX来运行old的文件~这里我分享DOXBOX的使用方法！

DOSBox 默认的 Z: 盘是系统模拟盘，不能直接放游戏。你需要用 mount 命令，把电脑上的一个真实文件夹映射成 DOSBox 里的 C 盘。

## 1.	准备游戏文件夹：

在你的电脑上（比如 D 盘或 E 盘）新建一个文件夹，专门用来存放 DOS 游戏，例如 D:\DOSGames。

## 2.	执行挂载命令：

在 DOSBox 窗口中，输入以下命令后按回车：

```text

mount c d:\dosgames
```

命令说明：mount 是挂载指令，c 代表 DOSBox 中的 C 盘，d:\dosgames 是你电脑上实际存放游戏的路径。

## 3.	切换盘符：

挂载成功后，输入 c: 并按回车，就切换到了虚拟 C 盘，提示符会变成 C:\>。

## 4.	进入游戏目录：

用 cd 命令进入具体游戏所在的子文件夹，例如：

```text
cd game_folder
```

## 5.	运行游戏：

输入游戏的启动文件（通常是 .exe 或 .bat 文件）名。可以先输入 dir 命令查看目录下有哪些文件，找到可执行文件后，输入文件名并按回车即可运行。
