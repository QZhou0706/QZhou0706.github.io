---
title: [计算机系统基础2. C语言拾遗(2):编程实践]
date: 2023-05-09 20:00:00
categories:
  - Computer System
tags:
  - 基础学习
---

**Preface:**

假设我们已经熟练使用各种C语言机制，原则上给需求就能搞定任何代码，但是结果并不是。本次课程就是教学怎么写代码才能从一个大型项目里存活下来

- 核心准则：编写可读代码

#### 核心准则：编写可读代码

[IOCCC'11 best self documenting program](https://www.ioccc.org/2011/hou/hou.c)

- 不可读 = 不可维护

```c
puts(usage: calculator 11/26+222/31
  +~~~~~~~~~~~~~~~~~~~~~~~~calculator-\
  !                          7.584,367 )
  +~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
  ! clear ! 0 ||l   -x  l   tan  I (/) |
  +~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
  ! 1 | 2 | 3 ||l  1/x  l   cos  I (*) |
  +~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
  ! 4 | 5 | 6 ||l  exp  l  sqrt  I (+) |
  +~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
  ! 7 | 8 | 9 ||l  sin  l   log  I (-) |
  +~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~(0
);
```

这是一段完全滥用 C 语言机制的代码，完全不可读。如果完全不可读，如果下次需要增加一个运算符，那么就极难进行维护。

这种代码或许平时不会写，但是一个现实中可能遇到的例子:

**人类不可读版 (STFW: clockwise/spiral rule)**

```c
void (*signal(int sig, void (*func)(int)))(int);
```

**人类可读版**

```c
typedef void (*sighandler_t)(int);
sighandler_t signal(int, sighandler_t);
```

上面这个函数实际上就是一个函数指针，返回值为和参数同类型的函数指针，但是可读版和不可读版读起来和改起来难度完全不同，下面的代码对人类更 friendly

#### 编写代码的准则：降低维护成本

为什么在编写一个比较长的代码的时候维护起来比较困难？长不是主要的问题，只是当代码的 size 增长之后，我们很难在回去看的时候明白每一个代码中的细节

>Programs are meant to be read by humans and only incidentally for computers to execute. — *D. E. Knuth*
>
>(程序首先是拿给人读的，其次才是被机器执行。)

##### 宏观

- 做好分解和解耦(现实世界也是这样管理复杂的系统的)
  - 大学中选课、住宿、食堂......

##### 微观

- “不言自明”
  - 通过**阅读代码**(直接阅读代码的文本)能理解一段程序是做什么的(specification)
- “不言自证”
  - 通过**阅读代码**能验证一段程序与 specification 的一致性，我阅读完代码之后，不仅知道代码是做什么的，还了解代码的输入输出是如何规范等等

##### 例子：实现数字逻辑电路

假想的数字逻辑电路

- 若干个 1-bit 边沿触发寄存器(X, Y, ...)
- 若干个逻辑门

我们怎么设计？

- 基本思路：状态（存储）模拟 + 计算模拟
  - 状态 = 变量
    - int X = 0, Y = 0;
  - 计算(while 循环)
    - X1 = !X && Y;
    - Y1 = !X && !Y;
    - X = X1; Y = Y1;

如果要增加一个触发器 Z ，我们就需要对代码作出一些改动

- 定义变量 Z, Z1

- 对 while 循环内的逻辑增加几行

如果项目不是这么简单，是一个很大的项目，以下的各种逻辑都用在各种文件中，我们要修改 such as 8 处，或许很容易出现各种编译会报错或者不报错(更可怕)的错误

**通用数字逻辑电路模拟器(cant'd)**

```c
#define FORALL_REGS(_)  _(X) _(Y)
#define LOGIC           X1 = !X && Y; \
                        Y1 = !X && !Y;

#define DEFINE(X)       static int X, X##1;
#define UPDATE(X)       X = X##1;
#define PRINT(X)        printf(#X " = %d; ", X);

int main() {
  FORALL_REGS(DEFINE);
  while (1) { // clock
    FORALL_REGS(PRINT); putchar('\n'); sleep(1);
    LOGIC;
    FORALL_REGS(UPDATE);
  }
}
```

##### 使用预编译：Good and Bad

Good

- 增加/删除寄存器只要修改一个地方
- 阻止一些编译错误
  - 忘记更新寄存器
  - 忘记打印寄存器
- “不言自明”还算不错

Bad

- 可读性变差(更不像 C 代码了)
  - “不言自证“还缺一些
- 给 IDE 解析带来一些困难

**更完整的实现：数码管显示**

[logisim.c](https://jyywiki.cn/pages/ICS/2020/demos/logisim.c) 和 [display.py](https://jyywiki.cn/pages/ICS/2020/demos/display.py)

- 也可以考虑增加诸如开关、UART等外设
- 原理无限接近数字电路课玩的 FPGA

FPGA？它不是万能的吗？？

- 我们能模拟它，是不是就能模拟计算机系统？

---

#### 例子：实现 YEMU 全系统模拟器

##### 最简单的计算机

**存储系统**

```
寄存器: PC, R0 (RA), R1, R2, R3 (8-bit)
内存：  16字节 (按字节访问)
```

指令集

````
       7 6 5 4   3 2   1 0
mov   [0 0 0 0] [ rt] [ rs]
add   [0 0 0 1] [ rt] [ rs]
load  [1 1 1 0] [   addr  ]
store [1 1 1 1] [   addr  ]
````

这就是一个 ISA ，前几位是一段二进制代码， CPU 接收到一段机器码，根据这段手册去匹配一些指令 mov, add, load, sotre......有“计算机系统“的感觉了？

- 它显然可以用数字逻辑电路实现(这个实现实际上和012是一样的)
- 不过我们**不需要在门层面实现它**
  - 接下来实现一个超级低配版 NEMU......

##### Y-Emulator (YEMU) 设计与实现

**存储模型：内存 + 寄存器(包含 PC)**

- 16 + 5 = 21 bytes = 168 bits
- 总共有 $2^{168}$种不同的取值
  - 任给一个状态，我们都能计算出 PC 处的命令，从而计算出下一个状态

理论上，任何计算机系统都是这样的**状态机**

- $(M,R)$构成了计算机系统的状态
- 32 GiB 内存有 $2^{274877906944}$种不同的状态
- 每个时钟周期，取出 $M[R[PC]]$ 的指令；执行；写回
  - 受制于物理实现(和功耗)的限制，通常每个始终周期只能改变少量的寄存器和内存的状态
  - (量子计算机颠覆了这个模型：同一时刻可以处于多个状态)

**存储**是计算机能实现“计算”的重要基础

- 寄存器(PC)、内存
- 这简单，用全局变量就好了！

```c
#include <stdint.h>
#define NREG 4
#define NMEM 16
typedef uint8_t u8; // 没用过 uint8_t？
u8 pc = 0, R[NREG], M[NMEM] = { ... };
```

- 建议 STFW(C 标准库) → bool 有没有？(Answer is "NO")
- 现代计算机系统：uint8_t == unsigned char
  - C Tips: 使用 unsigned int 避免潜在的 UB
    - -fwrapv 可以强制有符号整数溢出为 wraparound
  - C Quiz: 把指针转换成整数，应该用什么类型？(Answer is intptr_t in <stdint.h>)

**提升代码质量**

给寄存器名字？

```c
#define NREG 4
u8 R[NREG], pc; // 有些指令是用寄存器名描述的
#define RA 1    // BUG: 数组下标从0开始
... 
```

```c
enum { RA, R1, ..., PC };
u8 R[] = {
  [RA] = 0,  // 这是什么语法？？
  [R1] = 0,
  ...
  [PC]  = init_pc,
};

#define pc (R[PC]) // 把 PC 也作为寄存器的一部分
#define NREG (sizeof(R) / sizeof(u8))
```

使用 C 语言中的特性

- enum 添加寄存器
- 可以在数组中用 such as [RA] = 0 语法，避免在定义数组时多一个或者少一个 0 (人类难以避免犯这种错误)
  - 我们如何证明自己是正确的呢？
    - 对着第一行的 enum 中内容在下面 R[] 中逐个定义
- 用 sizeof(R) / sizeof(u8) 计算寄存器的大小，防止漏掉一些细节比如数组从 0 开始或者是新添加一个寄存器

**从一小带代码看软件设计**

软件里有很多隐藏的 dependencies (一些额外的、代码中没有体现和约束的“规则”)

- 一处改了，另一处忘了(例如加了一个寄存器但是忘记更新 NREG)
- 减少 dependencies → 降低代码耦合程度

```c
// breaks when adding a register
#define NREG 5 // 隐藏假设max{RA, RB, ... PC} == (NREG - 1)

// breaks when changing register size
#define NREG (sizeof(R) / sizeof(u8)) // 隐藏假设寄存器是8-bit

// never breaks
#define NREG (sizeof(R) / sizeof(R[0])) // 但需要R的定义

// even better (why?)
enum { RA, ... , PC, NREG }
```

最后一行中利用 C 语言的 enum 特性减少了 NREG 对寄存器数组的 dependencies

**PA 框架代码中的 CPU_state**

```c
struct CPU_state {
};

// C is not C++
// cannot declare "CPU_state state";

#define reg_l(index) (cpu.gpr[check_reg_index(index)]._32)
#define reg_w(index) (cpu.gpr[check_reg_index(index)]._16)
#define reg_b(index) (cpu.gpr[check_reg_index(index) & 0x3]._8[index >> 2])
```

对于复杂的情况，struct/union 是更好的设计

- 担心性能(check_reg_index)？
  - 在超强的编译器优化面前，不存在的

**YEMU:模拟指令执行**

在时钟信号驱动下，根据 $(M,R)$ 更新系统的状态

RISC 处理器(以及实际的 CISC 处理器实现)：

- 取值令(fetch): 读出 M[R[PC]] 的一条指令
- 译码(decode): 根据指令集规范解析指令的语义(顺便取出操作数)
- 执行(execute): 执行命令、运算后写回寄存器或内存

最重要的就是实现 idex()

- 在 PA 中最挣扎的地方

```c
int main() {
  while (!is_halt(M[pc])) {
    idex();
  }
}
```

**代码例子1**

```c
void idex() {
  if ((M[pc] >> 4) == 0) {
    R[(M[pc] >> 2) & 3] = R[M[pc] & 3];
    pc++;
  } else if ((M[pc] >> 4) == 1) {
    R[(M[pc] >> 2) & 3] += R[M[pc] & 3];
    pc++;
  } else if ((M[pc] >> 4) == 14) {
    R[0] = M[M[pc] & 0xf]; 
    pc++;
  } else if ((M[pc] >> 4) == 15) {
    M[M[pc] & 0xf] = R[0];
    pc++;
  }
}
```

**代码例子2**

是否好一些？

- 不言自明？不言自证？

```c
void idex() {
  u8 inst = M[pc++];
  u8 op = inst >> 4;
  if (op == 0x0 || op == 0x1) {
    int rt = (inst >> 2) & 3, rs = (inst & 3);
    if      (op == 0x0) R[rt]  = R[rs];
    else if (op == 0x1) R[rt] += R[rs];
  }
  if (op == 0xe || op == 0xf) {
    int addr = inst & 0xf;
    if      (op == 0xe) R[0]    = M[addr];
    else if (op == 0xf) M[addr] = R[0];
  }
}
```

**代码例子3**

```c
typedef union inst {
  struct { u8 rs  : 2, rt: 2, op: 4; } rtype;
  struct { u8 addr: 4,        op: 4; } mtype;
} inst_t;
#define RTYPE(i) u8 rt = (i)->rtype.rt, rs = (i)->rtype.rs;
#define MTYPE(i) u8 addr = (i)->mtype.addr;

void idex() {
  inst_t *cur = (inst_t *)&M[pc];
  switch (cur->rtype.op) {
  case 0b0000: { RTYPE(cur); R[rt]   = R[rs];   pc++; break; }
  case 0b0001: { RTYPE(cur); R[rt]  += R[rs];   pc++; break; }
  case 0b1110: { MTYPE(cur); R[RA]   = M[addr]; pc++; break; }
  case 0b1111: { MTYPE(cur); M[addr] = R[RA];   pc++; break; }
  default: panic("invalid instruction at PC = %x", pc);
  }
}
```

用到了 C 语言的一个特性：bit field，用一个类型的某些位来定义变量
