---
title: 操作系统06. 并发控制基础
date: 2023-04-23 20:00:00
categories: 
  - Operating System
tags:
  - 基础学习
---

**背景回顾**：虽然 “线程库” 入门简单，但多处理器编程 + 编译优化会给我们带来很多意想不到的惊喜。在编写多线程程序时，们必须放弃许多对顺序程序编程时的基本假设，这也是并发编程困难的原因。

<!--more-->

人类是 sequential creature ，大脑非常难以适应并发。但是可以创造一些新的手段让我们可以写出正确的并发程序。

- 手段：让并发”回退到“顺序执行
  - 标记若干块代码，使这些代码一定能按照某个顺序执行
  - 例如：我们可以安全地在块里记录执行的顺序

#### 回退到顺序执行：互斥

可以插入“神秘代码”，让其他所有代码不能并发，只让现在执行的东西顺序执行

- 由“神秘代码”领导的不会并发的代码（例如pure function       s）执行

```c
void Tsum() {
  stop_the_world();
  // 临界区 critical section
  sum++;
  resume_the_world();
}
```

Stop the world is possible

- Java有"stop the world GC"
- 单个处理器可以关闭终端
- 多处理器可以发送核间中断

##### 失败的尝试：

```c
int locked = UNLOCK;

void critical_section() {
retry:
  if (locked != UNLOCK) {
    goto retry;
  }
  locked = LOCK;

  // critical section

  locked = UNLOCK;
}
```

并发程序不能保证load+store的原子性，可以画状态机找到一种执行顺序证明上面的代码是不可以保证互斥的。

##### 更严肃地尝试：确定假设、设计算法

**假设：内存的读/写可以保证顺序、原子完成**

- val = atomic_load(ptr)
  - 看一眼某个地方的纸条（只能看到这个瞬间的字）
  - 刚看完可能就会被修改
- atomic_store(ptr, val)
  - 对应往某个地方“贴一张纸条”（闭眼盲贴，不管这个地方在发生什么）
  - 贴完一瞬间就可能被覆盖



**正确性不明的奇怪尝试（Peterson算法）**

类比为 A 和 B 争用厕所的包间

- 想进入包厢之前，A/B都先举起自己的旗子
  - A 往厕所门上贴上“B正在使用”的标签
  - B 往厕所门上贴上“A正在使用”的标签
- 然后，**如果对方举着旗子，而且厕所门上的名字是对方**，等待
  - 否则可以进入包厢
- 出包厢后，放下自己的旗子（完全不管门上的标签）

进入临界区的情况：

- 只有一个人举旗子，就可以直接进入
- 如果两个人同时举旗，由厕所门上的标签决定谁进
  - 手快🈶（被另一个人覆盖）、手慢🈚

一些具体的细节情况

- A 看到 B 没有举旗
  - B 一定不在临界区
  - 或者 B 想进但还没来得及把“A 正在使用”贴在门上
- A 看到 B 举旗
  - A 一定已经把旗子举起来了
  - and so on ......

Prove by brute-force

- 枚举状态机的全部状态（$PC_1,PC_2,x,y,trun$）CRAZY~~
- 但是手写还是容易写错——可执行的状态机模型有用了！

```c
void TA() { while (1) {
/* ❶ */  x = 1;
/* ❷ */  turn = B;
/* ❸ */  while (y && turn == B) ;
/* ❹ */  x = 0; } }

void TB() { while (1) {
/* ① */  y = 1;
/* ② */  turn = A;
/* ③ */  while (x && turn == A) ;
/* ④ */  y = 0; } }
```



#### 模型、模型检验与现实

##### 自动遍历状态空间的乐趣

可以帮助我们快速回答更多问题

- 如果结束后把门上的字条撕掉，算法还正确吗？
  - 在放下旗子之前撕
  - 在放下旗子之后撕
- 如果先贴标签再举旗，算法还正确吗？
- 我们有两个 “查看” 的操作
  - 看对方的旗有没有举起来
  - 看门上的贴纸是不是自己
  - 这两个操作的顺序影响算法的正确性吗？
- 是否存在 “两个人谁都无法进入临界区” (liveness)、“对某一方不公平” (fairness) 等行为？

> 把状态机看作图的节点，从一个状态机转化到另一个状态机实际上就是两个节点之间有一条有向边，每一个帧PC从一个节点到下一个节点（一个状态机转换为下一个状态机）。
>
> 因此类似上面的问题可以转换成图（状态空间）上的遍历问题了！

##### 回到假设（体现在模型）

- Atomic load & store
  - 读/写单个全局变量是“原子不可分割”的
  - 但这个假设在现代多处理器上并不成立

- 所以**实际上按照模型直接写Peterson算法应该是错的**？

"实现正确的Peterson算法"是合理需求，它一定能实现

- Compiler barrier/volatile 保证不被优化的前提下
  - 处理器提供特殊的指令保证可见性
  - 编译器提供 __sync_synchronize()函数
    - x86: mfence; ARM: dmb ish; RISC-V: fence rw, rw
    - 同时含有一个compiler barrier

#### 原子指令

##### 并发编程困难的解决

普通的变量读写在编译器+处理器的双重优化下行为变得复杂

```c
retry:
  if (locked != UNLOCK) {
    goto retry;
  }
  locked = LOCK;
```

解决方法：编译器和硬件共同提供”不可优化、不可打断“的指令

- “原子指令” + compiler barrier

之前的`sum.c`修改如下部分，就可以实现正确的求和

```c
for (int i = 0; i < N; i++)
  asm volatile("lock incq %0" : "+m"(sum));
```

“Bus lock”——从 80386 开始引入 (bus control signal)

![bus control signal](https://jyywiki.cn/pages/OS/img/80486-arch.jpg)

#### Take-away Messages

并发编程 “很难”：想要完全理解并发程序的行为，是非常困难的——我们甚至可以利用一个 “万能” 的调度器去帮助我们求解 NP-完全问题。因此，人类应对这种复杂性的方法就是退回到不并发。通过互斥实现 stop/resume the world，我们就可以使并发程序的执行变得更容易理解——而只要程序中 “能并行” 的部分足够多，串行化一小部分也并不会对性能带来致命的影响。
