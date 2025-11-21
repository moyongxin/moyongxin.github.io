+++
date = '2025-10-28T14:29:17+08:00'
draft = false
title = '写给新手看的：终端和 Shell 是啥？'
slug = 'terminal-n-shell'
author = 'qwertyuiop'
tags = ["开发环境"]
keywords = ["terminal", "shell", "命令行", "终端"]
readingTime = true
showFullContent = false
hideComments = false
+++

## 前言
某日我看到这么一篇[知乎回答](https://www.zhihu.com/question/19945828/answer/1965338461822247982)，发现自己写得文章也存在里面的问题，于是**痛定思痛**，决定写一篇开发中可能会经常用到的终端和 Shell 的入门文章。

## 先从终端讲起

### 从很久以前说起
电子计算机发展的较早期，计算机的体积非常庞大，操作也非常复杂。当时流行的模式是，多个用户通过多个名为**终端**的设备连接到一台大型计算机（主机）上进行操作，终端作为用户与系统交互的 I/O（输入/输出）设备存在，也就是说终端是用户与计算机之间的桥梁。比如下面图片展示的很出名的 [VT100 终端](https://en.wikipedia.org/wiki/VT100)：
{{< figure src="image.png" alt="This work is licensed under CC BY-SA 4.0" position="center" caption="This work is licensed under <a style=\"color: var(--background);\" href=\"https://creativecommons.org/licenses/by-sa/4.0/\">CC BY-SA 4.0</a>" captionPosition="center" >}}
终端一般有显示屏和键盘，用户通过键盘输入命令，终端将命令发送到主机进行处理，主机处理完后再将结果返回给终端显示出来。初期的终端设备多是基于字符界面的，后来随着图形用户界面（GUI）的发展，终端设备也逐渐支持图形显示([Sixel 协议](https://en.wikipedia.org/wiki/Sixel))。

### 那现在说的终端是啥？
随着计算机技术的发展，足够小的个人计算机出现了，终端作为独立的设备逐渐淡出我们的视野。现在我们提到的终端，通常指的是**终端模拟器**（Terminal Emulator），它是一种软件应用程序，模拟传统终端设备的功能，让用户能够在图形界面下使用字符界面（命令行）。具体来说，一个终端模拟器允许用户打开一个窗口（或者拆分出很多个窗口），在这个窗口中输入命令并与操作系统进行交互，它模拟旧时代的终端，实现字符界面的渲染功能，并且支持对各种终端功能的模拟（比如前面提到的 Sixel 协议，可以让命令行程序显示出图片）。
{{< figure src="image-1.png" alt="Windows Terminal" position="center" caption="Windows Terminal" captionPosition="center" >}}

## Shell 是啥

Shell，直接翻译过来是“壳”的意思，它的实际含义是充当用户与操作系统沟通的桥梁。按照这个定义，Windows 的图形界面本身（explorer.exe）就是 Shell，通过这个界面，我们可以查看部分正在运行的程序（任务栏和系统托盘）、管理文件、启动程序（双击可执行文件），所以它充当了用户与操作系统之间的桥梁。

当然，有图形界面的 Shell，那肯定也有命令行界面的 Shell。命令行界面的 Shell 主要通过文本命令与用户交互，用户输入命令，Shell 解析并执行这些命令，然后将结果返回给用户。Shell 可能还提供一些额外的功能，比如工作管理、命令历史记录等。

工作管理是啥呢，这里提供一个简单的例子来说明：
```bash
$ ./hello-world # 在前台执行 hello-world 程序
^C # 按下 Ctrl+C 中断当前前台任务
$ ./hello-world & # 在后台执行 hello-world 程序
$ jobs # 查看当前后台任务
[1]+  Running                 ./hello-world &
$ fg %1 # 将后台任务 1 切换到前台
# 按下 Ctrl+Z 暂停当前前台任务
[1]+  Stopped                 ./hello-world
$
```
可以看到，Shell 提供了在前台和后台执行任务的功能，并且可以随时挂起当前任务、恢复任务等。

## 终端和 Shell 的关系
终端和 Shell 是密切相关但又不同的两个概念。终端是用户与计算机交互的界面，而 Shell 是在这个界面中运行的程序，负责解释和执行用户输入的命令。终端提供了一个环境，让用户可以与 Shell 进行交互，而 Shell 则处理用户的命令并与操作系统进行通信。简单来说，终端就是那个窗口，而 Shell 则是你在窗口中使用的命令行解释器。


<!-- CC-BY-SA 4.0 -->
"[写给新手看的：终端和 Shell 是啥？](https://blog.moyongxin.top/posts/terminal-n-shell/index.md)" &copy; 2025 by [moyongxin](https://github.com/moyongxin) is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0)
{ style="color: color-mix(in srgb,var(--foreground) 65%,transparent); margin-bottom: 0px;" }
