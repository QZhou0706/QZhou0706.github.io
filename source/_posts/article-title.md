---
title: article title
date: 2023-04-20 14:50:14
tags: test
categories: test
---

```c++ Demo
#include <bits/stdc++.h>

using namespace std;

int main() {
	cout << "Hello World\n";
    int a = 1, b = 2;
    printf("a<=b\n");
    return 0;
}
```

$$\begin{equation} \label{eq1}
e=mc^2
\end{equation}$$

The famous matter-energy equation $\eqref{eq1}$ proposed by Einstein...

$$\begin{equation} \label{eq2}
\begin{aligned}
a &= b + c \\
  &= d + e + f + g \\
  &= h + i
\end{aligned}
\end{equation}$$

Equation $\eqref{eq2}$ is a multi-line equation.

$$\begin{align}
a &= b + c \label{eq3} \\
x &= yz \label{eq4} \\
l &= m - n \label{eq5}
\end{align}$$

There are three aligned equations: equation $\eqref{eq3}$, equation $\eqref{eq4}$ and equation $\eqref{eq5}$.

$$\begin{align}
-4 + 5x &= 2 + y \nonumber \\
w + 2 &= -1 + w \\
ab &= cb
\end{align}$$

$$x+1\over\sqrt{1-x^2} \tag{i}\label{eq_tag}$$

Equation $\eqref{eq_tag}$ use `\tag{}` instead of automatic numbering.
