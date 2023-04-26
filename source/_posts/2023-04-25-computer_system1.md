---
title: [计算机系统基础1. C语言拾遗(1):机制]
categories:	
  - Computer System
tags:
  - 基础学习
date: 2023-04-25 09:30:00

---



#### 概述

这一节主要是弥补一些程序设计基础课上不会讲到的一些关于 C 语言的比较重要的知识，帮助构筑整体知识框架，以便后续学习计算机系统能更加顺利一些。——总共学习了四个半小时。

<!--more-->
**以下几个问题值得思考：**

- 在IDE中，按一个建就能编译运行，中间的过程是什么？
  - 编译、连接
    - .c → 预编译 → .i → 编译 → .s → 汇编 → .o → 链接 → a.out
  - 加载执行
    - ./a.out
- 背后是通过调用命令行工具完成的
  - RTFM: gcc --help; man gcc
    - 控制行为的三个选项：-E, -S, -c
- 注意点：
  - 预热：编译、链接、加载到底做了什么？
  - RTFSC 时需要关注的 C 语言特性

#### 进入 C 语言之前：预编译

##### #include <>指令

以下代码有什么区别？

```c
#include <stdio.h>
#include "stdio.h"
```

为什么在没有安装库时会发生错误？

```c
#include <SDL2/SDL2.h>
```

很多人包括我在刚开始学习 C 语言时，肯定会问出一些问题：在引用stdio.h时到底是引用在哪里的文件？

可能在书/阅读材料上了解一些相关知识

- 但更好的办法是阅读命令的日志
- gcc --verbose a.c

使用gcc --verbose可以了解到很多内部的东西，例如：

这张图片是我的电脑上运行gcc --verbose a.c后包含的一段，这段信息显示了gcc在编译a.c文件时头文件引用的发生，分别找的是这些路径下的文件

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/gcc_verbose.png)



如果我有a.c和a.inc文件放在同一个路径下

```c a.inc
"hello world"
```

```c a.c
#include <stdio.h>
int main(){
printf(
#include "a.inc"
);
} 
```

然后使用gcc --verbose a.c后会发生什么？正确编译除了a.out，运行后输出hello world

如果把a.c改变一下，再编译就出现error了

```c a.c
#include <stdio.h>
int main(){
printf(
#include <a.inc>
);
} 
```

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230425090237647.png)

有一个命令选项-I可以添加gcc寻找#include <...>的路径，如果输入这些指令，编译通过，可以正常输出hello world。用gcc --verbose查看详细信息，发现#Include <...>多了一个path

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230425090737466.png)

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230425090859148.png)

##### 有趣的预编译

以下代码会输出什么？

- 为什么？

```c
#include <stdio.h>

int main() {
#if aa == bb
  printf("Yes\n");
#else
  printf("No\n");
#endif
}
```

运行结果是Yes，可以运行gcc -E a.c查看预编译代码，#include <stdio.h>中引用的内容太多了，这里只用到了其中的printf，可以用一行代码替换

```c printf
extern int printf (const char *__restrict __format, ...);
```

这是stdio.h中关于printf的定义，替换后再使用gcc -E a.c得到如下代码

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230425092836183.png)

其中那几行#开头的代码都被消失了，实际上是预编译先对代码进行词法上做处理，#if判断后面是否为true，如果是true则包含#if下的代码。C语言的很有意思的特性是所有预编译的变量，都不需要定义就可以使用，所以这里实际上是判断 $''==''$ 所以返回true，预编译后的代码包含printf("Yes\n")

还可以试试编译下面的代码，看看执行a.out后会输出什么，如果拿gcc -m32 a.c编译，又会输出什么？

```c 
extern int printf (const char *__restrict __format, ...);
int main() {
#ifdef __x86_64__
  printf("x86-64\n");
#else
  printf("x86\n");
#endif
#if aa == bb
  printf("Yes\n");
#else
  printf("No\n");
#endif
}
```

##### 宏定义与展开

宏展开：通过复制/粘贴改变代码的形态

- #include→ 粘贴文件

- aa,bb→ 粘贴符号

知乎问题：如何搞垮一个OJ？

下面这段代码会在一行输出$1^7$个a，会占用大量OJ资源，但是我在我们学校的OJ上试验了一下，输出超限后会被程序好像会被强制中断，所以现在搞不垮了。

```c
#define A "aaaaaaaaaa"
#define TEN(A) A A A A A A A A A A
#define B TEN(A)
#define C TEN(B)
#define D TEN(C)
#define E TEN(D)
#define F TEN(E)
#define G TEN(F)
int main() { puts(G); }
```

如何躲过Online Judge的关键字过滤？

```c
#define A sys ## tem
int main() {
  A("echo hello");
}
```

##的作用是在去除前后的所有空格，并且把前后非空格的两个部分按照连接起来，这样OJ就检测不到`system`这个单词了，但是实际上`system`已经起作用了，编译运行输出hello

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230425232043043.png)

如何毁掉一个身边用C语言的同学？

```c
#define true (__LINE__ % 16 != 0)
```

这段代码的意思是改变true的定义，true变为当代码的行数模16不等于0返回真，这就会引起非常大的歧义，如果偷偷趁同学不在，在他的电脑上的某一个头文件中插入这段代码，将在某一天对他造成毁灭性的打击！

宏展开：通过复制/粘贴改变代码的形态

- 反复粘贴，直到没有宏可以展开为止

例子：

```c X-macro
#define NAMES(X) \
  X(Tom) X(Jerry) X(Tyke) X(Spike)

int main() {
  #define PRINT(x) puts("Hello, " #x "!");
  NAMES(PRINT)
}
```

<u>其中#表示字符串化，#x相当于把x转换成字符串</u>

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230425233837276.png)

在发生实际编译之前

- 也称为元编程(meta-programming)
  - gcc的与处理器同样可以处理汇编代码
  - C++中的模板元编程；Rust的macros；...（这一块我基本看不懂，元编程和macros什么意思？）

Pros

- 提供灵活的用法 (X-macros)
- 接近自然语言的写法

Cons

- 破坏可读性 [IOCCC](https://www.ioccc.org/)、程序分析 (补全)、……

```c
#define L (
int main L ) { puts L "Hello, World" ); }
```

破坏了人类的直觉，造成括号不匹配，及其影响阅读

#### 编译与链接

##### 编译

一个不带优化的简易（理想）编译器

- C代码的连续一段总能找到对应的一段连续的机器指令
  - 因此大家说C是高级的汇编语言

```c
int foo(int n) {
  int sum = 0;
  for (int i = 1; i <= n; i++) {
    sum += i;
  }
  return sum;
}
```

编译之后对应的汇编语言大致如下（去除了一些不相干的代码）

```elm
foo:
.LFB0:
	pushq	%rbp
	movq	%rsp, %rbp
	movl	%edi, -20(%rbp)
	movl	$0, -8(%rbp)
	movl	$1, -4(%rbp)
	jmp	.L2
.L3:
	movl	-4(%rbp), %eax
	addl	%eax, -8(%rbp)
	addl	$1, -4(%rbp)
.L2:
	movl	-4(%rbp), %eax
	cmpl	-20(%rbp), %eax
	jle	.L3
	movl	-8(%rbp), %eax
	popq	%rbp
	ret
```

C语言代码有一一对应的汇编代码，汇编代码对应机器代码，上面的程序用objdump转换为机器代码：

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230425235744552.png)

##### 链接

将多个二进制目标代码拼接在一起

- C中成为编译单元(compilation unit)
- 甚至可以连接 C++, [rust](https://rust-embedded.github.io/book/interoperability/rust-with-c.html), ... 代码（不懂？_？）

```c a.c
int foo(int n) {
  int sum = 0;
  for (int i = 1; i <= n; i++) {
    sum += i;
  }
  return sum;
}
```

```c b.c
int foo(int);

int main() {
  printf("%d\n", foo(10));
}
```

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230426000753133.png)

这是b.c汇编的代码，怎么把a.o和b.o中的代码合到一起？那就要用到链接，a.c中定义了foo函数，b.c中申明foo函数，并且在main函数调用foo函数输出结果

输入gcc a.o b.o -static，运行./a.out输出结果55

可以使用objdump反汇编a.out可以在其中找到main和foo的内容

#### 加载：进入C语言的世界

##### C程序执行的两个视角

静态：C代码的连续一段总能找到一段连续的机器指令

动态：C代码执行的状态总能对应到机器的状态

- 源代码视角
  - 函数、变量、指针......
- 机器指令视角
  - 寄存器、内存、地址......

两个视角的共同之处：内存

- 代码、变量（源代码视角）=地址+长度（机器指令视角）
- （不太严谨地）内存=代码+数据+堆栈
  - 因此理解C程序执行最重要的就是<u>内存模型</u>

<u>本质上C语言中的所有的东西都是指针</u>，C语言的一切皆可取地址，常量、变量、函数、堆栈......在汇编中所有东西都是内存，两个刚好对应起来。

```c
void printptr(void *p) {
  printf("p = %p;  *p = %016lx\n", p, *(long *)p);
}
int x;
int main(int argc, char *argv[]) {
  int *p = (void *) 1;
  printptr(main);  // 代码
  printptr(&main);
  printptr(&x);    // 数据
  printptr(&argc); // 堆栈
  printptr(argv);
  printptr(&argv);
  printptr(argv[0]);
}
```

##### C Type System

**类型：**对一段内存的解读方式

- 非常“汇编“——没有class,polymorphism,type traits,...（看不懂）
- C里所有的数据都可以理解成是地址（指针）+类型（对地址的解读）

例子：

```c
int main(int argc, char *argv[]) {
  int (*f)(int, char *[]) = main;
  if (argc != 0) {
    char ***a = &argv, *first = argv[0], ch = argv[0][0];
    printf("arg = \"%s\";  ch = '%c'\n", first, ch);
    assert(***a == ch);
    f(argc - 1, argv + 1);
  }
}
```

这段代码可以画一张图去理解一下，其中比较有意思的是，函数作为指针、函数地址作为指针和*函数作为指针都是相同的

```c
int main() {
  printf("%p\n", main);
  printf("%p\n", &main);
  printf("%p\n", *main);
}
```

这段代码输出结果是完全相同的，说明函数的指针就是指向自身

有意思的函数指针：

- 当一个指针指向函数的时候，指针本身的地址没有改变，但是它指向一个指向函数的地址，函数的地址就是指向自身，所以一个上面例子中的f和(*f)甚至是(\*\*\*\*\*...\*f)都起到main函数的作用，但是(&f)就是取这个指针的地址。

```c
#include <stdio.h>
#include <assert.h>

int foo(int n) {
  int sum = 0;
  for (int i = 1; i <= n; i++)
    sum += i;
  return sum;
}

int main(int argc, char *argv[]) {
  int (*f)(int, char *[]) = main;

  printf("&foo = %p\n", &foo);
  printf("foo  = %p\n", foo);
  printf("*foo = %p\n", *foo);
  printf("&f   = %p\n", &f);
  printf("f    = %p\n", f);
  printf("*f   = %p\n", *f);
}
```

![](https://cdn.jsdelivr.net/gh/QZhou0706/picGoStorage@master/img/image-20230426230857144.png)

第一节终于上完了，比第一次看要学会了不少新的细节，这周不再看计算机系统基础，遇到知识点就回来看这篇文章吸收。

推荐文章：[Function Pointer in C](https://iq.opengenus.org/function-pointer-in-c/)
