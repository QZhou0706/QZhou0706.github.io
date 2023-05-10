---
title: 操作系统07. 并发控制互斥
date: 2023-04-27 11:00:00
categories:
  - Operating System
tags:
  - 基础学习
---

**背景回顾**：互斥 (Peterson 算法)：为了掌控并发程序的复杂行为，使程序退回到 “串行执行” 的 lock & unlock。

**本节内容：**现代多处理器系统上的互斥实现：

- 互斥问题的定义和假设
- 自旋锁
- 互斥锁和Futex

<!--more-->

#### 回顾：并发编程

##### 理解并发的工具

- 线程=人(大脑能完成局部存储和计算)
- 共享内存=物理世界(物理世界天生并行)
- 一切都是状态机(debugger&model checker)

![“躲进厕所锁上门，我就把全世界人锁在了厕所外“](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/wc.jpg)

##### 互斥问题：定义

互斥(mutual exclusion)，“互相排斥”

- 实现lock_t 数据结构和lock/unlock API:

```c
typedef struct {
  ...
} lock_t;
void lock(lock_t *lk);
void unlock(lock_t *lk);
```

一把“排他性”的锁——对于锁对象lk

- 如果某个线程持有锁，则其他线程的lock不能发挥(Safety)
- 如果多个线程执行lock时，至少有一个可以返回(Liveness)
- 能正确处理处理器乱序、宽松内存模型和编译优化

非常常见的一个bug：

一个线程用lock_A和unlock_A保护，其他进程如果也用lock_A和unlock_A锁的话就无法操作，但是如果其他线程用一个lock_B和unlock_B保护，那就可能造成lock_A和unlock_A之间与lock_B和unlock_B之间有重叠的数据可能会同时被两个线程访问到，就会产生错误的结果。

##### 互斥问题的经典算法

**Peterson算法**

- 包间、旗子和门上的字条
- 假设atomic load/store
  - 实现这个假设也不是非常容易

因此，<u>假设很重要</u>

- 不能同时读/写贡献内存(1960s)不是一个好的假设
  - Load(环顾四周)的时候不能写，“看一眼就把眼睛壁上”
  - Store(改变物理世界状态)的时候不能读，“闭着眼睛动手”
  - 这是《操作系统》
    - 更喜欢直观、简单、粗暴(稳定)、有效的解决方法

##### 实现互斥的基本假设

允许我们可以不管一切麻烦事的原子指令

```c
void atomic_inc(long *ptr);
int atomic_xchg(int val, int *ptr);
```


看起来是一个普通的函数，但假设：

- 包含一个原子指令
  - 指令的执行不能被打断
- 包含一个compiler barrier
  - 无论何种优化都不可雨果此函数
- 包含一个memory fence
  - 保证处理器在stop-the world前所有对内存的store都“生效”
  - 即对resume-the-world之后的load可见

```c Atomic Exchange实现
int xchg(int volatile *ptr, int newval) {
  int result;
  asm volatile(
    // 指令自带 memory barrier
    "lock xchgl %0, %1"
    : "+m"(*ptr), "=a"(result)
    : "1"(newval)
    // Compiler barrier
    : "memory"
  );
  return result;
}
```

> 原子指令是一个非常好的东西，我们程序的执行过程当中的任何时候，执行了原子指令，两条原子指令是有因果关系的，先执行的原子指令之前的所有内容在第二条原子指令之后都要是可见的。
>
> 在任何时候使用原子指令都是安全的，前提是原子指令是正确的。

#### 自选锁(Spin Lock)

##### 实现互斥

```c
void lock(lock_t *lk);
void unlock(lock_t *lk);
```

一个科学家的思维应该是：考虑更多更根本的问题

- 我们可以设计出怎么样的原子指令？
  - 它们的表达能力如何？
- 计算机硬件可以提供比“一次load/store”更强的原子性吗？
  - 如果硬件困难，软件/编译器可以么？

##### 自旋锁：用xchg实现互斥

在厕所门口放一个桌子(共享变量)

- 初始时放着🔑

自旋锁(Spin Lock)

- 想上厕所的同学(一条xchg指令)
  - Stop the world
  - 看一眼桌子上有什么(🔑 或 🔞)
  - 把 🔞 放到桌上 (覆盖之前有的任何东西)
  - Resume the world
  - 期间看到🔑才可以进厕所，否则重复
- 出厕所的同学
  - 把🔑放到桌上

```c
int table = YES;

void lock() {
retry:
  int got = xchg(&table, NOPE);
  if (got == NOPE)
    goto retry;
  assert(got == YES);
}

void unlock() {
  xchg(&table, YES);  // 为什么不是 table = YES; ?
}
```

在xchg的假设下简化实现

- 包含一个原子指令
- 包含一个compiler barrier
- 包含一个memory fence

```c sum-spinlock.c
#include "thread.h"

#define N 100000000
#define M 10

long sum = 0;

int xchg(int volatile *ptr, int newval) {
  int result;
  asm volatile(
    "lock xchgl %0, %1"
    : "+m"(*ptr), "=a"(result)
    : "1"(newval)
    : "memory"
  );
  return result;
}

int locked = 0;

void lock() {
  while (xchg(&locked, 1)) ;
}

void unlock() {
  xchg(&locked, 0);
}

void Tsum() {
  long nround = N / M;
  for (int i = 0; i < nround; i++) {
    lock();
    for (int j = 0; j < M; j++) {
      sum++;  // Non-atomic; can optimize
    }
    unlock();
  }
}

int main() {
  assert(N % M == 0);
  create(Tsum);
  create(Tsum);
  join();
  printf("sum = %ld\n", sum);
}
```

```c thread.h
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <stdatomic.h>
#include <assert.h>
#include <unistd.h>
#include <pthread.h>

#define NTHREAD 64
enum { T_FREE = 0, T_LIVE, T_DEAD, };
struct thread {
  int id, status;
  pthread_t thread;
  void (*entry)(int);
};

struct thread tpool[NTHREAD], *tptr = tpool;

void *wrapper(void *arg) {
  struct thread *thread = (struct thread *)arg;
  thread->entry(thread->id);
  return NULL;
}

void create(void *fn) {
  assert(tptr - tpool < NTHREAD);
  *tptr = (struct thread) {
    .id = tptr - tpool + 1,
    .status = T_LIVE,
    .entry = fn,
  };
  pthread_create(&(tptr->thread), NULL, wrapper, tptr);
  ++tptr;
}

void join() {
  for (int i = 0; i < NTHREAD; i++) {
    struct thread *t = &tpool[i];
    if (t->status == T_LIVE) {
      pthread_join(t->thread, NULL);
      t->status = T_DEAD;
    }
  }
}

__attribute__((destructor)) void cleanup() {
  join();
}
```

上面用自旋锁优化的sum-spinlock.c可以编译成功，输出正确结果

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230427122620575.png)



可以验证，无论是O2优化还是O1优化都可以输出正确的结果，自旋锁目前为止是可以处理编译优化的。lock和unlock内没有compiler barrier里面的代码是可以优化的

