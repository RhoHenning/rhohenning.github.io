---
title: 浅谈进位制
date: 2019-08-29
categories:
- C/C++
tags:
- 进位制
---

## 普通进位制

对于一个 $k(k\geqslant 2)$ 进制的数 $\cdots a_2a_1a_0.a_{-1}a_{-2}\cdots$，其每一位的值都满足 $a_i\in\{0,1,2,\cdots,k-1\}$，该数在数值上等于

$$\sum_ik^ia_i$$

<!-- more -->

### 整数部分

若该数在 $k$ 进制下是 $n$ 位正整数 $a_{n-1}a_{n-2}\cdots a_2a_1a_0$，则其数值是一个关于 $k$ 的 $n-1$ 次多项式：

$$\sum_{i=0}^{n-1}k^ia_i$$

可用秦九韶算法求出，也可用一个变量更新 $k$ 的幂，时间复杂度均为 $O(n)$，不过讨论数位的顺序相反。

$$\begin{aligned}
\sum_{i=0}^{n-1}k^ia_i&=a_0+ka_1+k^2a_2+\cdots+k^{n-1}a_{n-1}\\
&=a_0+k(a_1+k(a_2+k(\cdots+ka_{n-1})))
\end{aligned}$$

```cpp
int base_k(int *a, int n, int k) {
  int ans = 0;
  for(int i = n-1; i >= 0; i--) ans = ans*k+a[i];
  return ans;
}
int base_k2(int *a, int n, int k) {
  int ans = 0, b = 1;
  for(int i = 0; i < n; i--) ans += a[i]*b, b *= k;
  return ans;
}
```

反过来，若要将一个给定的正整数 $x$ 用 $k$ 进制表示，即 $x=a_0+k(a_1+k(a_2+k(\cdots+ka_{n-1})))$，则有

$$\begin{aligned}
x\bmod k&=a_0\\
\left\lfloor\dfrac{x}{k}\right\rfloor&=a_1+k(a_2+k(\cdots+ka_{n-1}))
\end{aligned}$$

令 $x'=\left\lfloor\dfrac{x}{k}\right\rfloor$，则又有 $x'\bmod k=a_1$；令 $x''=\left\lfloor\dfrac{x'}{k}\right\rfloor$，则 $x''\bmod k=a_2$……不断重复该过程，可求出每一位的值 $a_i$。若要单独求第 $i$ 位的值，可根据上述过程推得

$$a_i=\left\lfloor\dfrac{x}{k^i}\right\rfloor\bmod k,\quad i=0,1,2,\cdots$$

```cpp
void base_k(int *a, int &n, int k, int x) {
  n = 0; while(x) a[n++] = x%k, x /= k;
}
```

---

### 小数部分

若该数在 $k$ 进制下是小数，提取其小数部分 $0.a_{-1}a_{-2}\cdots a_{-m}$，则其数值为

$$\sum_{i=1}^{m}k^{-i}a_{-i}=\dfrac{1}{k}\sum_{i=0}^{m-1}\left(\dfrac{1}{k}\right)^ia_{-i-1}$$

其中包含一个关于 $\dfrac{1}{k}$ 的 $m-1$ 次多项式，可如法炮制。

$$\dfrac{1}{k}\sum_{i=0}^{m-1}\left(\dfrac{1}{k}\right)^ia_{-i-1}=\dfrac{1}{k}\left(a_{-1}+\dfrac{1}{k}\left(a_{-2}+\dfrac{1}{k}\left(\cdots+\dfrac{1}{k}a_{-m}\right)\right)\right)$$

```cpp
double base_k(int *a, int m, int k) {
  double ans = 0;
  for(int i = m; i >= 1; i--) ans = ans/k+a[i];
  return ans;
}
double base_k2(int *a, int m, int k) {
  double ans = 0, b = 1;
  for(int i = 1; i <= m; i--) b /= k, ans += a[i]*b;
  return ans;
}
```

若要将 $(0,1)$ 内的小数 $y$ 用 $k$ 进制表示，即 $y=\dfrac{1}{k}\left(a_{-1}+\dfrac{1}{k}\left(a_{-2}+\dfrac{1}{k}\left(\cdots+\dfrac{1}{k}a_{-m}\right)\right)\right)$，则有

$$\begin{aligned}
\lfloor ky\rfloor&=a_{-1}\\
\{ ky\}&=\dfrac{1}{k}\left(a_{-2}+\dfrac{1}{k}\left(\cdots+\dfrac{1}{k}a_{-m}\right)\right)
\end{aligned}$$

令 $y'=\{ ky\}$，则又有 $\lfloor ky'\rfloor=a_{-2}$；令 $y''=\{ ky'\}$，则 $\lfloor ky''\rfloor=a_{-3}$……不断重复该过程，可求出每一位的值。求小数部分时通常会限制小数部分的位数 $m$。

```cpp
void base_k(int *a, int m, int k, double y) {
  for(int i = 1; i <= m; i++) a[i] = (y *= k), y -= (int)y;
}
```

---

## 其它进制系统

下面讨论一些特殊的进制系统。在这些进制下，求一个用该进制表示的数字的值，仍然可以用 $k^ia_i$ 求和；但要将其他进制下的数字用该进制表示，需要采用其它的方法。

### 平衡三进制

普通三进制每一位的数值是 $0,1,2$，平衡三进制每一位的数值是 $0,1,-1$。$-1$ 在数中用 $\mathrm{T}$ 表示。

由于引入了 $-1$，平衡三进制可以直接表示负数。并且由于 $1,-1$ 符号相反，直接将数中的 $1$ 和 $\mathrm{T}$ 互换，得到的数就是原数的相反数。

平衡三进制与普通三进制可通过改写相互转换。若普通三进制下第 $i$ 位是 $2$，则将其对应的数值 $3^i\times 2$ 改写成 $3^{i+1}\times 1+3^i\times(-1)$，即把 $2$ 改成 $\mathrm{T}$ 再向前进一位。反之，若平衡三进制下第 $i$ 位是 $-1$，则将其对应的数值 $3^i\times(-1)$ 改写成 $3^{i+1}\times(-1)+3^i\times 2$，即把 $\mathrm{T}$ 改成 $2$ 再向前借一位。若最高位没有位可借，则该数是负数，需要先改写其相反数。

因此，把一个数字用平衡三进制表示时，先将其表示成普通三进制，再从低位到高位讨论，每遇到 $2$，就把 $2$ 改成 $\mathrm{T}$，然后向前进一位。不过要注意下一位可能会由 $2$ 变成 $3$，此时要照常进位。

```cpp
void balanced_ternary(int *a, int &n, int x) {
  n = 0; while(x) a[n++] = x%3, x /= 3;
  for(int i = 0; i < n; i++) if(a[i] >= 2) a[i] -= 3, a[i+1]++;
  if(a[n]) n++;
}
```

### 不含零的进制

普通的 $k$ 进制由数字 $0,1,2,\cdots,k-1$ 组成，而不含零的 $k$ 进制由数字 $1,2,3,\cdots,k$ 组成。不含零的进制只能表示正整数（这里暂不考虑小数的情况）。一个典型的例子是 Excel 表格的列序号，即不含零的 $26$ 进制。

若普通 $k$ 进制下的第 $i$ 位是 $0$，则其一定不是最高位，设第 $i+1$ 位是 $a(1\le a\le k)$，其对应的数值 $k^{i+1}a$ 可改写成 $k^{i+1}(a-1)+k^i\cdot k$，即把第 $i$ 位改成 $k$ 再向前借一位。若 $a-1=0$，则重复此过程。

但是，将一个数 $x$ 用不含零的 $k$ 进制表示时，没必要先求出普通 $k$ 进制表示，再修改等于 $0$ 的数位。在求第 $i$ 个 $k$ 进制位时，$x$ 已经变成了 $x'=\left\lfloor\dfrac{x}{k^i}\right\rfloor$，此时若第 $i$ 位等于 $0$，要将第 $i+1$ 位减 $1$，相当于 $x$ 减去 $k^{i+1}$，也即 $x'$ 减去 $k$，也即 $x''=\left\lfloor\dfrac{x'}{k}\right\rfloor$ 减去 $1$。

```cpp
void base_k_nzero(int *a, int &n, int k, int x) {
  n = 0; while(x) {
    a[n++] = x%k, x /= k;
    if(!a[n-1]) x--, a[n-1] = k;
  }
}
```
