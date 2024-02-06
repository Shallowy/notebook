# Chapter 3 插值与多项式近似 Interpolation and Polynomial Approximation
目标：给定插值点 $(x_0,f(x_0)),\ (x_1,f(x_1)),\ ,...,(x_n,f(x_n))$，构造多项式函数 $g(x)\approx f(x)$.

## 3.1 拉格朗日插值法 Lagrange Interpolation
### 拉格朗日插值多项式
将最终得到的插值多项式记为 $P_n(x)=a_0+a_1x+...+a_nx^n$ 满足 $P_n(x_i)=y_i,\ i=0,1,...,n$，且 $x_i$ 互不相同。

构造 $L_{n,i}(x)$ 使得 $L_{n,i}(x_j)=\delta_{ij}=\begin{cases} 1,& i=j\\ 0,& i\neq j \end{cases}$ .

则只需令 $P_n(x)=\sum\limits_{i=0}^nL_{n,i}(x)y_i$，就有 $P_n(x_i)=y_i$.

观察发现，$L_{n,i}$ 有 $n$ 个根 $x_0,...,\widehat{x_i},...,x_n$(即除 $x_i$ 外). 

$$\Rightarrow\ L_{n,i}(x)=C_i(x-x_0)...\widehat{(x-x_i)}...(x-x_n)=C_i\prod\limits_{j=0,j\neq i}^n(x-x_j)$$

又有 $L_{n,i}(x_i)=1$,

$$\Rightarrow\ C_i=\prod_{j=0,j\neq i}^n\frac{1}{x_i-x_j}.$$

得到**拉格朗日基 (Lagrange Basis)**

$$L_{n,i}(x)=\prod_{j=0,j\neq i}^n\frac{x-x_j}{x_i-x_j}$$

和**拉格朗日插值多项式 (Lagrange Interpolating Polynomial)**

$$P_n(x)=\sum\limits_{i=0}^nL_{n,i}(x)y_i$$

!!! note "拉格朗日插值多项式的唯一性"
    $n+1$ 个插值点插出来的 $n$ 阶拉格朗日多项式唯一。
    
    $n+1$ 个点唯一确定一个 $n$ 阶多项式，可用反证法证明（假设存在两个多项式，将其相减，导出一个不高于 $n$ 阶的多项式有 $n+1$ 个零点，矛盾）。


### 余项分析 Remainder Analysis

假设 $a\leq x_0<x_1<...<x_n\leq b$ 且 $f\in C^{n+1}[a, b]$.

考虑截断误差，余项

$$R_n(x)=f(x)-P_n(x)$$

可知 $R_n(x)$ 有至少 $n+1$ 个根

$$\Rightarrow\ R_n(x)=K(x)\prod\limits_{i=0}^n(x-x_i)$$

目标是得到 $K(x)$ 的范围。

对任意 $x\neq x_i(i=0,...,n)$，定义 $[a,b]$ 上关于 $t$ 的函数

$$g(t)=R_n(t)-K(x)\prod\limits_{i=0}^n(t-x_i)$$

则 $g(x)$ 至少有 $n+2$ 个根：$x_0,...x_n,x\ \ $  $\ \ \Rightarrow\ \ $  $\ g^{(n+1)}(\xi_x)=0,\ \xi_x\in(a,b)$.

又因为 $P^{(n+1)}(\xi_x)=0$，所以等式两边求 $n+1$ 阶导，得到

$$\begin{aligned}
&0=f^{(n+1)}(\xi_x)-K(x)\times (n+1)! \\
\\
\Rightarrow\ &R_n(x)=\frac{f^{(n+1)}(\xi_x)}{(n+1)!}\prod\limits_{i=0}^n(x-x_i)
\end{aligned}$$

由于 $\xi_x$ 的值难以确定，一般估计一个 $f^{(n+1)}$ 的上界 $|f^{(n+1)}(x)|\leq M_{n+1}$，则

$$|R_n(x)|\leq\frac{M_{n+1}}{(n+1)!}\prod\limits_{i=0}^n|x-x_i|$$

!!! note "推论"
    由误差上界式可知，拉格朗日多项式对阶数 $\leq n$ 的多项式的拟合都是精准的，因为有 $f^{(n+1)}(x)=0$.