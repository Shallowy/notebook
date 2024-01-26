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

$$L_{n,i}=\prod_{j=0,j\neq i}^n\frac{x-x_j}{x_i-x_j}$$

和**拉格朗日插值多项式 (Lagrange Interpolating Polynomial)**

$$P_n(x)=\sum\limits_{i=0}^nL_{n,i}(x)y_i$$