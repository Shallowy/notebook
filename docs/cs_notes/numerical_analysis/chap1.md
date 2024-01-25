# Chapter 1 数学基础 Mathematical Preliminaries

## 1.1 误差
### 舍入误差 roundoff error 与 截断误差 truncation error
`舍入误差 roundoff error`
:   循环小数截断产生的误差，与有效位数有关 | replacing a number with its floating-point form

`截断误差 truncation error`
:   仍掉余项产生的误差，与计算量有关 | using a truncated, or finite, summation to approximate the sum of an infinite seires

???+ example

    #### Approximate $\bold{\int_0^1e^{-x^2}dx}$.

    ---

    Use Taylor expansion:

    $$\begin{aligned}\int_0^1e^{-x^2}dx &= \int_0^1(1-x^2+\frac{x^4}{2!}-\frac{x^6}{3!}+\frac{x^8}{4!}-...)dx\\ &= \underbrace{1-\frac{1}{3}+\frac{1}{2!}\times\frac{1}{5}-\frac{1}{3!}\times\frac{1}{7}}_{S_4}+\underbrace{\frac{1}{4!}\times\frac{1}{9}-...}_{R_4}\end{aligned}$$

    Take
    
    $$\begin{aligned}\int_0^1e^{-x^2}dx\approx S_4 &= 1-\frac{1}{3}+\frac{1}{10}-\frac{1}{42}\\ &\approx 1-0.333+0.100-0.024 \\ &= 0.74\underline{3}\end{aligned}$$

    $\bold{\Rightarrow\textbf{roundoff error}< 0.0005\times 2 = 0.001}$.
    
    And

    $$\begin{aligned}|R_4| = |\frac{1}{4!}\times\frac{1}{9}-\frac{1}{5!}\times\frac{1}{11}+...|<\frac{1}{4!}\times\frac{1}{9}<0.005\end{aligned}$$

    $\bold{\Rightarrow\textbf{truncation error}< 0.005}$.

    $$\bold{\therefore|\textbf{total error}|<0.001+0.005=0.006}$$

### 舍入 rounding 与 截断 chopping
`舍入 rounding`
:   四舍五入.

`截断 chopping`
:   直接丢弃.

Given a real number $y = 0.d_1d_2...d_kd_{k+1}d_{k+2}...\times 10^n$,

$$fl(y) = \begin{cases}& 0.d_1d_2...d_k\times 10^n, &\text{  //chopping} \\ chop(y+5\times 10^{-(k+1)+n})=& 0.\delta_1\delta_2...\delta_k\times 10^n &\text{  //rounding}\end{cases}$$

### 绝对误差 absolute error 与 相对误差 relative error
Let $p^*$ be an approximation to $p$.

`绝对误差 absolute error`
:   $|p-p^*|$

`相对误差 relative error`
:   $\frac{|p-p^*|}{|p|}$

### 有效数字 significant digits
`有效数字 significant digits`
:   The number $p^*$ is said to approximate $p$ to $t$ **significant digits** if $t$ is the largest nonnegative integer for which $\frac{|p-p^*|}{|p|}<5\times 10^{-t}$.
:    此定义和我们平常理解的有效数字一致。

---

???+ note "有效位数的损失"
    运算过程中有效位数会损失。如相近的两个小数相减会减少有效位数：

    $a_1 = 0.1234\underline{5},\ a_2 = 0.1234\underline{6}$ 均有5位有效数字，但 $a_1 - a_2 = 0.00001$ 只有1位有效数字。

## 1.2 减小误差的方法
!!! example
    **讨论：用3-digit arithmetic 估计 $f(x) = x^3-6.1x^2+3.2x+1.5$ 在 $x=4.71$处的值。**

    ---

    直接依次计算各项并列表，结果如下：

    ||$x$|$x^2$|$x^3$|$3.2x$|$6.1x^2$|
    |::|::|::|::|::|::|
    |**Exact**|4.71|22.1841|104.487111|15.072|135.32301|
    |**Three-digit(rounding)**|4.71|22.2|105.|15.1|135.|
    |**Three-digit(chopping)**|4.71|22.1|104.|15.0|134.|

    - $\text{exact}=-14.263899,\ \text{chopping}=-13.5,\ \text{rounding}=-13.4;$
    - $\text{relative error(chopping)}=5\%,\ \text{relative error(rounding)}=6\%.$

### 秦九韶算法 Horner's Method
!!! note "原理"
    乘除转成加减，可以减小误差：

    $$|ab+a\varepsilon_b+b\varepsilon_a+\varepsilon_a\varepsilon_b|\Rightarrow|a+\varepsilon_a+b+\varepsilon_b|$$

通过不断提取多项式中的自变量 $x$ 来减少乘法次数，提高精度。

!!! example
    $$f(x)=x^3-6.1x^2+3.2x+1.5=((x-6.1)x+3.2)x+1.5$$

    - $\text{chopping}=-14.2,\ \text{rounding}=-14.3;$
    - $\text{relative error(chopping)}=0.45\%,\ \text{relative error(rounding)}=0.25\%.$

    精度大幅提高。

## 1.3 收敛 Convergence
### 算法的稳定性
`稳定 stable`
:   初值的微小改变只导致结果的微小改变。

`不稳定 unstable`
:   初值的微小改变导致结果的巨大改变。

`条件稳定 conditionally stable`
:   只对部分特定的初值稳定。

???+ example
    **Evaluate $I_n=\frac{1}{e}\int_0^1x^ne^xdx,\ n=0,1,2,...$**

    ---

    根据关系式 $I_n=1-nI_{n-1}$，有以下两种方法：

    1. 直接积分计算近似值 $I_0^*\approx I_0$ 作为初值，并迭代得到 $I_n$ 。出人意料的是，结果为正负交替的发散数列。因为这样得到的误差 $|E_n|=|I_n-I_n^*|=|(1-nI_{n-1})-(1-nI_{n-1}^*)|=n|E_{n-1}|=...=n!|E_0|$，算法**不稳定**，初值的微小误差会被大幅放大。
    2. 反代得到 $I_{n-1}=\frac{1}{n}(1-I_n)$.由 $\frac{1}{e(n+1)}<I_n<\frac{1}{n+1}$，直接取 $I_N^*=\frac{1}{2}[\frac{1}{e(N+1)}+\frac{1}{N+1}]\approx I_N$ 为初值进行迭代，得到剩下的 $I_n$.同理可得 $|E_n|=\frac{1}{N(N-1)...(n+1)}|E_N|$，算法**稳定**。

### 误差增长类型
`线性增长 linear`
:   $E_n\approx CE_0$，其中 $C$ 为与 $n$ 无关的常数。线性增长是可接受的。

`指数增长 exponential`
:   $E_n\approx C^nE_0$，其中 $C>1$ 且与 $n$ 无关。指数增长是不可接受的。

---

???+ note
    线性增长误差一般不可避免，指数增长误差要尽量避免。

### 收敛率 the rate of convergence
假设 $lim_{h\rightarrow 0}G(h)=0,\ lim_{h\rightarrow 0}F(h)=L$. 如果存在正常数 $K$ 使得 $|F(h)-L|\leq K|G(h)|$ 对足够小的$h$成立，则可记为$F(h)=L+O(G(h))$.**收敛率**可记为 $O(G(h))$.

一般取 $G(h)=h^p(p>0)$.