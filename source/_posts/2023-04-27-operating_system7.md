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

#### 自旋锁(Spin Lock)

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

##### 无锁算法(了解)

**更强大的原子指令**

Compare and exchange("test and set")

- (lock) compxchg SRC, DEST

```c
TEMP = DEST
if accumulator == TEMP:
    ZF = 1
    DEST = SRC
else:
    ZF = 0
    accumulator = TEMP
```

- 🤔 看起来没有复杂多少，又好像复杂了很多
  - 学编程/操作系统”纸面理解“是不行的
  - 一定要写代码加深印象
    - 对于这个例子：我们可以列出“真值表”

```c cmpoxchg.c
#include <stdio.h>
#include <assert.h>

int cmpxchg(int old, int new, int volatile *ptr) {
  asm volatile(
    "lock cmpxchgl %[new], %[mem]"
      : "+a"(old), [mem] "+m"(*ptr)
      : [new] "S"(new)
      : "memory"
  );
  return old;
}

int cmpxchg_ref(int old, int new, int volatile *ptr) {
  int tmp = *ptr;  // Load
  if (tmp == old) {
    *ptr = new;  // Store (conditionally)
  }
  return tmp;
}

void run_test(int x, int old, int new) {
  int val1 = x;
  int ret1 = cmpxchg(old, new, &val1);

  int val2 = x;
  int ret2 = cmpxchg_ref(old, new, &val2);

  assert(val1 == val2 && ret1 == ret2);
  printf("x = %d -> (cmpxchg %d -> %d) -> x = %d\n", x, old, new, val1);
}

int main() {
  for (int x = 0; x <= 2; x++)
    for (int old = 0; old <= 2; old++)
      for (int new = 0; new <= 2; new++)
        run_test(x, old, new);
}
```

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230510124338922.png)

#### 在操作系统上实现互斥

##### 自旋锁的缺陷

性能问题(1)

- 除了进入临界区的线程，其他处理器上的线程都在**空转**
- 争抢锁的处理器越多，利用率越低
  - 4 个 CPU 运行 4 个 sum-spinlock 和 1 个 OBS
    - 人一时刻都只有一个 sum-atomic 在有效计算
  - 均分 CPU, OBS 就分不到 100% 的 CPU 了

性能问题(2)

- 持有自选锁的线程**可能被操作系统切换出去**
  - 操作系统不"感知"线程在做什么
  - (但为什么不能呢？)
- 实现 100% 的资源浪费

##### Scalability: 性能的新维度

>同一份计算任务，时间 (CPU cycles) 和空间 (mapped memory) 会随处理器数量的增长而变化。

用自旋锁实现 sum++ 的性能问题

- 严谨的统计很难
  - CPU 动态功耗
  - 系统的其他进程
  - 超线程
  - NUMA
  - ......
  - [Benchmarking crimes](https://gernot-heiser.org/benchmarking-crimes.html)

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/spinlock-scalability.jpg)

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230510130507698.png)

线程越多，效率越低，用本地代码去测试也有这种情况，为什么会发生这种情况？一个很重要的原因是现代的处理器是有缓存的，当我们想要读 sum 或者写 sum 都是从一级缓存中读出的，但是当另一个线程想要读不在这个处理器缓存中的 sum 时，就需要去另一个 CPU 的缓存中拉下来，传的过程就会产生时间，很有趣的 Scalability 的讨论

##### 自旋锁的使用场景

1. 临界区几乎不“拥堵”
2. 持有自旋锁时禁止执行流切换

使用场景**(唯一)**：**操作系统内和的并发数据结构（短临界区）**

- 操作系统可以关闭中断和抢占(仔细读起来这个行为是不合理的)
  - 保证自旋锁的持有者在很短的时间内可以释放锁
- (如果是虚拟机呢...😂)
  - PAUSE 指令会处罚 VM Exit
- 但依旧很难做好
  - [An analysis of Linux scalability to many cores](https://www.usenix.org/conference/osdi10/analysis-linux-scalability-many-cores) (OSDI'10)

##### 实现线程 + 长临界区的互斥

>作业那么多，与其干等作业发布，不如把自己 (CPU) 让给其他作业 (线程) 执行？

“让”不是 C 语言代码可以做到的 (C 代码只能执行命令)

- 但有一种特殊的指令：syscall
- 把锁的实现放到操作系统里就好了
  - syscall(SYSCALL_lock, &lk);
    - 试图获得 1k，但如果失败，就切换到其他进程
  - syscall(SYSCALL_unlock, &1k);
    - 释放 1k，如果有等待锁的线程就唤醒

操作系统 = 更衣室管理员

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/locker.jpg)

- 先到的人(线程)
  - 成功获得手环，进入游泳馆
  - \*1k = 🔒，系统调用直接返回
- 后到的人(线程)
  - 不能进入游泳馆，排队等待
  - 线程放入等待队列，执行线程切换(yield)
- 洗完澡出来的人(线程)
  - 交还手环给管理员；管理员把手环再交给排队的人
  - 如果等待队列不空，从等待队列中取出一个线程允许执行
  - 如果等待队列为空，\*1k = ✅
- **管理员 (OS) 使用自旋锁确保自己处理手环的过程是原子的**

#### 关于互斥的一些分析

自旋锁(线程直接共享 locked)

- 更快的 fast path
  - xchg 成功→立即进入临界区，开小很小
- 更慢的 slow path
  - xchg 失败→浪费 CPU 自旋等待

互斥锁(通过系统调用访问 locked)

- 更经济的 slow path
  - 上锁失败线程不再占用 CPU
- 更慢的 fast path
  - 即使上锁成功也需要进出内核 (syscall)

---

为了实现现代多处理器系统上的互斥，我们首先需要理解 “原子操作” (例如 `atomic_xchg`) 的假设：

1. 操作本身是原子的、看起来无法被打断的，即它真的是一个 “原子操作”；
2. 操作自带一个 compiler barrier，防止优化跨过函数调用。这一点很重要——例如我们今天的编译器支持 Link-time Optimization (LTO)，如果缺少 compiler barrier，编译优化可以穿过 volatile 标记的汇编指令；
3. 操作自带一个 memory barrier，保证操作执行前指令的写入，能对其他处理器之后的 load 可见。

在此假设的基础上，原子操作就成为了我们简化程序执行的基础机制。通过自旋 (spin)，可以很直观地实现 “轮询” 式的互斥。而为了节约共享内存线程在自旋上浪费的处理器，我们也可以通过系统调用请求操作系统来帮助现成完成互斥。

以上就是本节的所有内容，中间因为各种原因隔了十多天才全部看完，这段间隔也让我发现了前导知识的缺陷，线程库的那一章被我跳过了，我感觉需要去把前面的多处理器编程补回来，然后这篇文章中的一些链接都看一下，大致就能更深入地理解并发了。
