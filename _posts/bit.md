---
title: 位运算及其应用
date: 2018-10-30
categories:
- C/C++
tags:
- 位运算
---

## 数据存储

### 整数存储

将正整数 $x$ 储存在 $n$ 位内存空间中，即转化成 $n$ 位二进制整数 $a_{n-1}a_{n-2}\cdots a_2a_1a_0$，该二进制表示称为 $x$ 的**原码**。将原码中的 $0$ 和 $1$ 互换，得到**反码**。反码加一得到**补码**。若 $x\ge 2^n$，则第 $n$ 位及之后的二进制位自动溢出，相当于储存的数值为 $x\bmod 2^n$。

<!-- more -->

不难发现原码和反码相加等于

$$\overbrace{111\cdots 11}^n=\sum_{i=0}^{n-1}2^i=2^n-1$$

于是原码和补码相加等于 $2^n\equiv 0\pmod{2^n}$。因此补码可视为原码在模 $2^n$ 意义下的相反数。

无符号整数类型只能表示非负整数，用原码储存。$n$ 位无符号整数类型表示的数据范围为 $[0,2^n)$。

有符号整数类型的最高位为**符号位**，符号位为 $0$ 表示正整数或 $0$，符号位为 $1$ 表示负整数。正整数和 $0$ 储存为原码，负整数储存为其相反数的补码。$n$ 位有符号整数类型表示的数据范围为 $\left[-2^{n-1},2^{n-1}\right)$。

将超出类型范围的整数存入 $n$ 位整数类型，其二进制表示对应的数值始终与原数在模 $2^n$ 意义下同余。

### 浮点数存储

浮点数类型的最高位为**符号位**，符号位为 $0$ 表示正数或 $0$，符号位为 $1$ 表示负数。其余二进制位被划分为两个部分，分别表示**指数**和**尾数**。指数为 $e$、尾数为 $a$ 的浮点数的数值为 $\pm a\times 2^e$。

尾数 $a$ 的二进制表示的整数部分为 $1$，并且只储存其小数部分。

指数 $e$ 只能是整数。若用 $n$ 个二进制位表示指数，则指数 $e$ 存储为 $e+2^{n-1}-1$。这样做的目的是不使用符号位又能存储负数。但有两个特殊情况：
- 若表示指数的二进制位全为 $0$，则指数 $e=-2^{n-1}+1$。此时尾数 $a$ 的整数部分视为 $0$。
- 若表示指数的二进制位全为 $1$，则整个浮点数表示特殊值。
  - 若表示尾数的二进制位全为 $0$，则整个浮点数表示无穷大，符号由最高位决定。
  - 若表示尾数的二进制位不全为 $0$，则整个浮点数表示 NaN。

单精度浮点数（$32$ 位）用 $8$ 位表示指数，$23$ 位表示尾数。双精度浮点数（$64$ 位）用 $11$ 位表示指数，$52$ 位表示尾数。

---

## 位运算

本文中的位运算只针对整数；移位运算借用了符号 $\ll,\gg$。

### 与

两个整数作**与**运算，对于每个二进制位，都是 $1$ 才取 $1$。即 $x\operatorname{and}y=z$，当且仅当 $x,y,z$ 的第 $i$ 个二进制位 $x_i,y_i,z_i$ 满足

$$z_i=\begin{cases} 1,&x_i=1\land y_i=1\\ 0,&x_i\neq 1\lor y_i\neq 1 \end{cases} =\left\lfloor\dfrac{x_i+y_i}{2}\right\rfloor=x_iy_i$$

与运算常用来判断整数的奇偶性，若 $x$ 是奇数，则 $x\operatorname{and}1=1$，若 $x$ 是偶数，则 $x\operatorname{and}1=0$。

### 或

两个整数作**或**运算，对于每个二进制位，只要有 $1$ 就取 $1$。即 $x\operatorname{or}y=z$，当且仅当 $x,y,z$ 的第 $i$ 个二进制位 $x_i,y_i,z_i$ 满足

$$z_i=\begin{cases} 1,&x_i=1\lor y_i=1\\ 0,&x_i\neq 1\land y_i\neq 1 \end{cases} =\left\lceil\dfrac{x_i+y_i}{2}\right\rceil=\left\lfloor\dfrac{x_i+y_i+1}{2}\right\rfloor$$

### 异或

两个整数作**异或**运算，对于每个二进制位，同为 $0$，异为 $1$。即 $x\oplus y=z$，当且仅当 $x,y,z$ 的第 $i$ 个二进制位 $x_i,y_i,z_i$ 满足

$$z_i=\begin{cases} 1,&x_i\neq y_i\\ 0,&x_i=y_i \end{cases} =(x_i+y_i)\bmod 2$$

异或运算满足交换律和结合律，其逆运算是其本身。有 $x\oplus x=0$，$x\oplus 0=0\oplus x=x$。

### 取反

一个整数**取反**就是将每一个二进制位取反。即 $\operatorname{not}x=y$，当且仅当 $x,y$ 的第 $i$ 个二进制位 $x_i,y_i$ 满足

$$y_i=\begin{cases} 1,&x_i=0\\ 0,&x_i=1 \end{cases} =1-x_i$$

反码就是原码取反得到的结果。由于补码等于反码加 $1$，可得

$$\operatorname{not}x=-x-1$$

另外，有$\operatorname{not}\operatorname{not}x=x$。

### 移位

$x\ll y$ 表示把 $x$ 的每个二进制位向左移动 $y$ 位，移动造成的最右边的空位用 $0$ 补足，最左边的数溢出。

$x\gg y$ 表示把 $x$ 的每个二进制位向右移动 $y$ 位，移动造成的最左边的空位用 $0$ （无符号类型）或符号位（有符号类型）补足，最右边的数溢出。

位运算的优先级普遍较低，使用时需要注意。取反作为单目运算符，比所有双目运算符的优先级都高。移位运算比比较运算优先级高，比加减运算优先级低。与、或、异或优先级由高到低，比比较运算优先级低，比逻辑运算优先级高。

---

## 位运算的应用

### 数学运算

显然，当 $x\ge 0$ 时，有如下乘法、除法、取模的关系：

$$\begin{aligned}
2^ix&=x\ll i\\
\left\lfloor\dfrac{x}{2^i}\right\rfloor&=x\gg i\\
x\bmod 2^i&=x\operatorname{and}\left((1\ll i)-1\right)
\end{aligned}$$

```cpp
int mulpow(int x, int i) { return x<<i; }
int divpow(int x, int i) { return x>>i; }
int modpow(int x, int i) { return x&(1<<i)-1; }
```

当 $x<0$ 时，除法严格向下取整，因为负数的存储涉及到取反，原本向零取整的方向也跟着反向；负数取模的值非负，因为负数与非负数相与，符号位的结果为 $0$。

### 数位变换

$lowbit(x)$ 表示 $x$ 的二进制表示的最后一个 $1$ 表示的数：

$$lowbit(x)=x\operatorname{and}(\operatorname{not}x+1)=x\operatorname{and}-x$$

```cpp
int lowbit(int x) { return x&-x; }
```

下面是针对 $x$ 的第 $i$ 个二进制位的变换。

- 获取该位的数值：$(x\gg i)\operatorname{and}1$
- 获取该位表示的数：$x\operatorname{and}(1\ll i)$
- 取反：$x\oplus(1\ll i)$
- 置为$1$：$x\operatorname{or}(1\ll i)$
- 置为$0$：$x\operatorname{and}\operatorname{not}(1\ll i)$

```cpp
int findbit(int x, int i) { return x>>i&1; }
int findbit2(int x, int i) { return x&1<<i; }
int flipbit(int x, int i) { return x^1<<i; }
int setbit(int x, int i, int d) { return d ? x|1<<i : x&~1<<i; }
```

下面是针对 $x$ 的前 $i$ 个二进制位的变换。

- 获取：$x\operatorname{and}((1\ll i)-1)$
- 取反：$x\oplus((1\ll i)-1)$
- 置为$1$：$x\operatorname{or}((1\ll i)-1)$
- 置为$0$：$x\operatorname{and}\operatorname{not}((1\ll i)-1)$

```cpp
int findbits(int x, int i) { return x&(1<<i)-1; }
int flipbits(int x, int i) { return x^(1<<i)-1; }
int setbits(int x, int i, int d) { return d ? x|(1<<i)-1 : x&~((1<<i)-1); }
```

下面是针对 $x$ 的二进制低位连续数字的变换。

- 把二进制低位的连续的 $0$ 都变成 $1$：$x\operatorname{or}(x-1)$
- 把二进制低位的连续的 $1$ 都变成 $0$：$x\operatorname{and}(x+1)$

```cpp
int setbits(int x, int d) { return d ? x|x-1 : x&x+1; }
```

### 字符变换

大写字母的 ASCII 码范围为 $[65,90]$，第 $5$ 个二进制位都为 $0$；小写字母的 ASCII 码范围为 $[97,122]$，和对应的大写字母刚好相差 $2^5=32$。因此有对字母字符 $c$ 的变换：

- 大写变小写，小写变大写：$c\oplus 32$
- 强制变小写：$c\operatorname{or}32$
- 强制变大写：$c\operatorname{and}\operatorname{not}32$

```cpp
int flipalpha(int c) { return c^32; }
int setlower(int c) { return c|32; }
int setupper(int c) { return c&~32; }
```

数字字符的 ASCII 码范围为 $[48,57]$，第 $4,5$ 个二进制位都为 $1$，而 $48=2^4+2^5$。因此有对数字字符 $d$ 的变换：

- 字符变数字，数字变字符：$d\oplus 48$
- 强制变字符：$d\operatorname{or}48$
- 强制变数字：$d\operatorname{and}\operatorname{not}48$

```cpp
int flipdigit(int d) { return d^48; }
int setchar(int d) { return d|48; }
int setdigit(int d) { return d&~48; }
```

### 集合操作

设全集 $U=\{a_0,a_1,a_2,\cdots,a_{n-1}\}$，其有 $2^n$ 个子集，分别对应 $[0,2^n)$ 内的整数。整数的第 $i$ 个二进制位为 $1$，表示该整数对应的集合包含 $a_i$。因此之前对二进制位的变换可用于对集合元素的变换。此外还可以得到（大写字母表示集合，小写字母表示对应的整数）

$$\begin{aligned}
S\cap T&\leftrightarrow s\operatorname{and}t\\
S\cup T&\leftrightarrow s\operatorname{or}t\\
\complement_US&\leftrightarrow s\oplus u\\
S\setminus T&\leftrightarrow s\operatorname{and}\operatorname{not}t
\end{aligned}$$

还有非常经典的枚举 $S$ 的非空子集 $T$ 的操作：初始时令 $t=s$，设 $T$ 是当前枚举的 $S$ 的子集，枚举的下一个子集表示为 $t'=(t-1)\operatorname{and}s$。

```cpp
for(int t = s; t; t = t-1&s);
```

下面说明其正确性。

若 $S$ 为全集，即 $s=2^n-1$，其各个二进制位都为 $1$，则 $t'=(t-1)\operatorname{and}s=t-1$ 恰好枚举了 $[0,2^n)$ 内的所有数，也即 $S$ 的所有子集。

容易发现 $t-1$ 只会将 $t$ 的二进制表示中最右边的 $1$ 及其右边的 $0$ 全部取反。因为 $T$ 是 $S$ 的子集，对于 $t$ 中为 $1$ 的二进制位，$s$ 中的相应位置一定为 $1$。而对于 $t$ 中为 $0$ 的二进制位，如果 $s$ 中的相应位置为 $0$，$(t-1)\operatorname{and}s$ 的结果中这些位置仍为 $0$。因此 $(t-1)\operatorname{and}s$ 只会影响 $s$ 中相应位置为 $1$ 的二进制位，并且只会把 $t$ 的这些二进制位中最右边的 $1$ 及其右边的 $0$ 全部取反。如果把这些二进制位提取出来拼成一个新数，则这个数在初始时各个二进制位都为 $1$，且以上过程就是将这个数不断减去 $1$。根据前面的分析，这样操作恰好枚举了 $s$ 的所有子集。

枚举子集的时间复杂度为 $O(2^n)$，枚举子集的子集的时间复杂度为

$$O\left(\sum_{i=0}^nC_n^i2^i\right)=O(3^n)$$

---

## 完备性

函数 $f(x_1,x_2,\cdots,x_n)$ 满足 $x_1,x_2,\cdots,x_n$ 的值以及函数值都为 $0,1$。给定 $x_1,x_2,\cdots,x_n$ 的所有 $2^n$ 种取值情况对应的函数值，用位运算写出$f$的表达式。

**分析：** 设函数 $g(x_1,x_2,\cdots,x_n)$ 满足只有当 $x_1=y_1,x_2=y_2,\cdots,x_n=y_n$ 时函数值为 $1$，$y_1,y_2,\cdots,y_n$ 已知。

$x_i=y_i$ 的真假性可用 $x_i$ 表示：

$$\begin{aligned}
x_i=y_i&\leftrightarrow\operatorname{not}x_i,&\quad y_i=0\\
x_i=y_i&\leftrightarrow x_i,&\quad y_i=1
\end{aligned}$$

$g(x_1,x_2,\cdots,x_n)=1$ 即所有的 $x_i=y_i$ 同时成立，将对应的 $x_i$ 或 $\operatorname{not}x_i$ 全部与起来即可。

$f(x_1,x_2,\cdots,x_n)=1$ 可能有不同的 $x_1,x_2,\cdots,x_n$ 的值 $y_1,y_2,\cdots,y_n$ 都能使之成立，将对应的 $g(x_1,x_2,\cdots,x_n)$ 全部或起来即可。

例如，按如下方式定义 $f(x_1,x_2,x_3)$：

$$\begin{aligned}
f(0,0,0)&=0&f(0,0,1)&=1\\
f(0,1,0)&=1&f(0,1,1)&=0\\
f(1,0,0)&=0&f(0,0,1)&=0\\
f(1,1,0)&=0&f(1,1,1)&=1
\end{aligned}$$

- 对于 $f(0,0,1)=1$，定义 $g_1(x_1,x_2,x_3)=\operatorname{not}x_1\operatorname{and}\operatorname{not}x_2\operatorname{and}x_3$，满足 $g_1(0,0,1)=1$，其它取值 $g_1(x_1,x_2,x_3)=0$。
- 对于 $f(0,1,0)=1$，定义 $g_2(x_1,x_2,x_3)=\operatorname{not}x_1\operatorname{and}x_2\operatorname{and}{not}x_3$，满足 $g_2(0,1,0)=1$，其它取值 $g_2(x_1,x_2,x_3)=0$。
- 对于 $f(1,1,1)=1$，定义 $g_3(x_1,x_2,x_3)=x_1\operatorname{and}x_2\operatorname{and}x_3$，满足 $g_3(1,1,1)=1$，其它取值 $g_3(x_1,x_2,x_3)=0$。

于是

$$\begin{aligned}
f(x_1,x_2,x_3)&=g_1(x_1,x_2,x_3)\operatorname{or}g_2(x_1,x_2,x_3)\operatorname{or}g_3(x_1,x_2,x_3)\\
&=(\operatorname{not}x_1\operatorname{and}\operatorname{not}x_2\operatorname{and}x_3)\operatorname{or}(\operatorname{not}x_1\operatorname{and}x_2\operatorname{and}\operatorname{not}x_3)\operatorname{or}(x_1\operatorname{and}x_2\operatorname{and}x_3)
\end{aligned}$$

```cpp
int f(int x1, int x2, int x3) {
  return (~x1&~x2&x3)|(~x1&x2&~x3)|(x1&x2&x3);
}
```

类似地，也可设函数 $h(x_1,x_2,\cdots,x_n)$ 满足只有当 $x_1=z_1,x_2=z_2,\cdots,x_n=z_n$ 时函数值为 $0$。这样按照上面的方法得到的函数，其函数值与正确值相反，把整个函数取反即可。此时还可根据德摩根定理消去最外层的取反。上面的例子的另一个表达式：

```cpp
int f(int x1, int x2, int x3) {
  return (x1|x2|x3)&(x1|~x2|~x3)&(~x1|x2|x3)&(~x1|x2|~x3)&(~x1|~x2|x3);
}
```

若对于任意的 $n$ 和任意的函数 $f$，都可以只用某些位运算写出 $f$ 的表达式，称这些位运算具有**完备性**。

根据上面的分析，与、或、取反三种运算具有完备性。

由于

$$x\operatorname{or}y=\operatorname{not}(\operatorname{not}x\operatorname{and}\operatorname{not}y)$$

所以与、取反两种运算具有完备性。

由于

$$x\operatorname{and}y=\operatorname{not}(\operatorname{not}x\operatorname{or}\operatorname{not}y)$$

所以或、取反两种运算具有完备性。

### 与非、或非

定义**与非**运算，与非运算的结果与“与”相反。

$$x\uparrow y=\operatorname{not}(x\operatorname{and}y)$$

定义**或非**运算，或非运算的结果与“或”相反。

$$x\downarrow y=\operatorname{not}(x\operatorname{or}y)$$

由于

$$\begin{aligned}
\operatorname{not}x&=\operatorname{not}(x\operatorname{and}x)=x\uparrow x\\
x\operatorname{and}y&=\operatorname{not}(x\uparrow y)=(x\uparrow y)\uparrow (x\uparrow y)
\end{aligned}$$

所以与非运算具有完备性。

由于

$$\begin{aligned}
\operatorname{not}x&=\operatorname{not}(x\operatorname{or}x)=x\downarrow x\\
x\operatorname{or}y&=\operatorname{not}(x\downarrow y)=(x\downarrow y)\downarrow (x\downarrow y)
\end{aligned}$$

所以或非运算具有完备性。

---

## 只出现 $p$ 次的数

下面解决一类问题，这类问题通常对时间或空间有严格的限制。

读入一些数，其中有一个数 $x$ 只出现 $p$ 次，其它数出现的次数都是 $q$ 的倍数，$p<q$ 且 $p,q$ 互质。找出 $x$。

**分析1：** 这里将异或的概念扩展到 $k$ 进制。在 $k$ 进制下 $x\oplus y=z$，当且仅当 $x,y,z$ 的第 $i$ 个 $k$ 进制位 $x_i,y_i,z_i$ 满足

$$z_i=(x_i+y_i)\bmod k$$

这样会得到跟二进制异或类似的结论：

$$\begin{aligned}
\overbrace{x\oplus x\oplus\cdots\oplus x}^k&=0\\
x\oplus 0=0\oplus x&=x
\end{aligned}$$

在这个问题中，将所有数在 $q$ 进制下异或，则出现次数为 $q$ 的倍数的数全部抵消，得到的结果就是 $p$ 个 $x$ 异或的结果。记这个结果为 $y$，根据异或的定义，$x,y$ 的第 $i$ 个 $q$ 进制位 $x_i,y_i$ 满足

$$y_i=px_i\bmod q$$

可用扩展欧几里得算法求出 $[0,q)$ 内的解 $x_i$。由于 $p,q$ 互质，这样的解只会有一个。解出每一位的数值 $x_i$ 后即可求出 $x$。

代码中虽然用数组 $a$ 储存了所有的数，但在实际题目中由于空间限制需要边读入边处理。

```cpp
int t[30];
int ext_gcd(int a, int b, int &x, int &y) {
  if(!b) { x=1, y=0; return a; }
  int d = ext_gcd(b, a%b, y, x);
  y -= a/b*x; return d;
}
int find(int *a, int n, int p, int q) {
  for(int i = 0; i < n; i++)
    for(int j = 0; a[i]; j++)
      t[j] = (t[j]+a[i]%q)%q, a[i] /= q, j++;
  int ans = 0, b = 1, x, y, d = ext_gcd(p,q,x,y);
  for(int j = 0; j < 30; j++) ans += (t[j]*x/d%q+q)%q*b, b *= q;
  return ans;
}
```

上面的代码用数组 $t$ 储存异或结果，$t_i$ 为第 $i$ 个 $q$ 进制位，因此 $t$ 数组的大小为 $\lfloor\log_q\max\{ a_i\}\rfloor+1$。

**分析2：** 在异或运算的定义中，即使 $x_i,y_i,z_i$ 是 $x,y,z$ 在 $m(m\neq k)$ 进制下的数位，只要异或运算满足

$$z_i\equiv(x_i+y_i)\bmod k\pmod m$$

仍然有

$$\overbrace{x\oplus x\oplus\cdots\oplus x}^k=0$$

因此可以直接将所有数的二进制位相加后模 $q$，出现次数为 $q$ 的倍数的数也会全部抵消。并且由于 $x$ 出现了 $p$ 次，取模后的结果只能是 $0,p$，可直接视为 $0,1$，作为答案的二进制位。总结起来就是，把所有数的二进制表示当成 $q$ 进制表示，在 $q$ 进制下异或起来，再将异或结果当成二进制表示，作为答案。这样甚至不需要知道 $p$ 具体的数值。这里 $t$ 数组的大小为 $\lfloor\log_2\max\{ a_i\}\rfloor+1$。

```cpp
int t[30];
int find(int *a, int n, int q) {
  for(int i = 0; i < n; i++)
    for(int j = 0; j < 30; j++) t[j] += a[i]>>j&1;
  int ans = 0;
  for(int j = 0; j < 30; j++) ans |= (t[j]%q>0)<<j;
  return ans;
}
```

**分析3：** 还有一种更流行但更晦涩的写法：把所有数的二进制表示当成 $q$ 进制表示，用 $t_i$ 的第 $j$ 个二进制位表示异或结果的第 $j$ 个 $q$ 进制位数值的第 $i$ 个二进制位。其出发点是沿用二进制的位运算，用多个数的同一个二进制位表示一个 $q$ 进制位，此时 $t$ 数组的大小为 $s=\lfloor\log_2q\rfloor+1$。

只关注第 $j$ 个二进制位，则 $t$ 数组就是异或结果的第 $j$ 个 $q$ 进制位的二进制表示。每读入一个数，将其二进制表示当成 $q$ 进制表示，把第 $j$ 个 $q$ 进制位（$0$ 或 $1$）加到异或结果的第 $j$ 个 $q$ 进制位的二进制表示上，即用 $t$ 数组模拟二进制加法。

两个二进制位 $a_i,b_i$ 对应相加，当前位结果为 $(a_i+b_i)\bmod 2=a_i\oplus b_i$，进位为 $\left\lfloor\dfrac{a_i+b_i}{2}\right\rfloor=a_i\operatorname{and}b_i$。但进位的部分不能直接加到下一位。这里的加法只有一个二进制位的空间，需要及时处理所有的进位。不难发现，若第 $i$ 位需要向第 $i+1$ 位进位，则第 $i$ 位为 $1$ 且需要加上 $1$，加上的 $1$ 要么来自第 $i-1$ 位的进位，要么来自另一个加数。而这里的加数只有在为 $1$ 时才会加到最低位上。因此可以推知，第 $i$ 位加 $1$ 的充要条件为所有的较低位以及加数都为 $1$。

代码中从高位到低位讨论二进制位，是为了避免较低的二进制位加 $1$ 后影响对较高的二进制位是否加 $1$ 的判断。

```cpp
for(int i = s-1; i >= 0; i--) {
  int d = a[k];
  for(int j = 0; j < i; j++) d &= t[j];
  t[i] ^= d;
}
```

同时，相加后的结果还要对 $q$ 取模。由于每次最多加 $1$，只需判断结果是否为 $q$，是则置为 $0$。可以用前面的方法构造函数 $f(t_0,t_1,t_2,\cdots,t_{n-1})$，只有当 $t$ 是 $q$ 的二进制表示时函数值为 $0$，然后用函数值与结果。代码中的 $f$ 实际上是 $\operatorname{not}f$。

```cpp
int f = ~0;
for(int j = 0; j < s; j++) f &= q>>j&1 ? t[j] : ~t[j];
for(int j = 0; j < s; j++) t[j] &= ~f;
```

最终只需判断每一个 $q$ 进制位是否非零，把该位数值的二进制位 $t_0,t_1,t_2,\cdots,t_{n-1}$ 或起来即可。

```cpp
int t[30];
int find(int *a, int n, int q) {
  int s = log2(q)+1;
  for(int k = 0; k < n; k++) {
    for(int i = s-1; i >= 0; i--) {
      int d = a[k];
      for(int j = 0; j < i; j++) d &= t[j];
      t[i] ^= d;
    }
    int f = ~0;
    for(int j = 0; j < s; j++) f &= q>>j&1 ? t[j] : ~t[j];
    for(int j = 0; j < s; j++) t[j] &= ~f;
  }
  int ans = 0;
  for(int j = 0; j < s; j++) ans |= t[j];
  return ans;
}
```

最后这种方法通常在 $q$ 为确定的常数时使用，代码会很简洁。如 $q=3$ 时代码如下：

```cpp
int find(int *a, int n) {
  int t0 = 0, t1 = 0, f;
  for(int k = 0; k < n; k++)
    t1 ^= t0&a[k], t0 ^= a[k], f = t0&t1, t0 &= ~f, t1 &= ~f;
  return t0|t1;
}
```
