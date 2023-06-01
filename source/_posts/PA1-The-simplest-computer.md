---
title: PA1:The simplest computer
date: 2023-04-28 23:00:00
categories:
  - PA实验
---

**Preface:**PA或许比较困难，我想记录一下途中遇到的困难，以及我是如何解决，在这个实验的过程中我希望能培养一下我先前比较弱的RTFSC能力。

<!--more-->

#### BUGs：

- 在`开始愉快的PA之旅之前`中，运行fceux-am时报错**(UNcompleted)**：

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230502191442346.png)

如下修改过后，fceux-am能运行，声音和游戏都能运行，如果切换成字符模式可以正常玩，但是没有画面，至今不知道怎么解决，也不知道哪里有问题。

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230502195059434.png)

---

- 运行risv32-NEMU时发现报错如下

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/Screenshot from 2023-04-28 23-11-19.png)

看了calloc的manual，发现是给gdb_conn \*指针申请空间的时候申请了sizeof(strcut gdb_conn \*)大小的内存，实际上内存应该申请的是sizeof(strcut gdb_conn)大小

修改后成功编译运行risv32-NEMU，欢迎界面：

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230428232022537.png)

---

- x86-CPU-state中寄存器部分需要修改

做出如下修改：

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230502192612669.png)

成功运行x86-NEMU，欢迎界面：

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230502193010338.png)

---

