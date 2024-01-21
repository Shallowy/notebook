# Chapter 1 数学基础 Mathematical Preliminaries

## 误差
### 舍入误差 roundoff error 与 截断误差 truncation error
`舍入误差 roundoff error`
:   循环小数截断产生的误差，与有效位数有关 | replacing a number with its floating-point form

`截断误差 truncation error`
:   仍掉余项产生的误差，与计算量有关 | using a truncated, or finite, summation to approximate the sum of an infinite seires

---

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

---

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

## 减小误差的方法
**讨论：用3-digit arithmetic 估计 $f(x) = x^3-6.1x^2+3.2x+1.5$ 在 $x=4.71$处的值。**

||$x$|$x^2$|$x^3$|$3.2x$|$6.1x^2$|
|::|::|::|::|::|::|
|**Exact**|4.71|22.1841|104.487111|15.072|135.32301|
|**Three-digit(rounding)**|4.71|22.2|105.|15.1|135.|
|**Three-digit(chopping)**|4.71|22.1|104.|15.0|134.|

### 秦九韶算法 Horner's Method