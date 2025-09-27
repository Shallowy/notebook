# RoPE 旋转式位置编码 Rotary Positional Encoding

Sinusoidal 位置编码给出了一种通过绝对位置编码的方式实现相对位置编码的思路。然而它的位置编码只加在输入端的 token 表示上，很难在后续的上层模块中持续地提供位置信息。（应该还有别的缺点，但我没想明白）

如果能直接配合 Transformer 的注意力机制来实现位置编码，或许就能一定程度地解决上述问题。

---

## 1. RoPE 理论推导

既然注意力的核心计算就是内积，那我们刚好可以利用内积来实现位置编码。具体而言，我们考虑在位置 $m$ 对应的 $q$， 和位置 $n$ 对应的 $k$ 上，分别添加绝对位置信息： $$ {q_m}^\prime = f(q_m, m),\ {k_n}^\prime = f(k_n, n) $$

进行内积运算后，满足以下恒等关系： $$ {q_m}^\prime \cdot {k_n}^\prime = f(q_m, m) \cdot f(k_n, n) = g(q_m, k_n, m - n) $$

注意 $f$ 是向量，$g$ 是标量。我们从最简单的，$q$ 和 $k$ 都是 2 维开始考虑，并将向量 $\begin{bmatrix} x \\ y \end{bmatrix}$ 看作复数 $x + yi$. 因为内积和复数乘法满足 $ x\cdot y = Re(x \overline{y}) $，其中 $ \overline{y} $ 为 $y$ 的共轭复数，所以有 $$ Re(f(q_m, m) \overline{f(k_n, n)}) = g(q_m, k_n, m - n) $$

我们作**假设1. 复数 $f(q_m, m) \overline{f(k_n, n)}$ 的乘积复数（不仅是实部，也包括虚部）也是关于 $q_m, k_n, m - n$ 的函数**：

$$ \begin{aligned} && r_f(q_m, m) e^{i\theta_f(q_m, m)} \cdot r_f(k_n, n) e^{-i\theta_f(k_n, n)} &= r_g(q_m, k_n, m - n) e^{i\theta_g(q_m, k_n, m - n)} \\  \Rightarrow && r_f(q_m, m) r_f(k_n, n) e^{i(\theta_f(q_m, m) - \theta_f(k_n, n))} &= r_g(q_m, k_n, m - n) e^{i\theta_g(q_m, k_n, m - n)} \end{aligned} $$

$$ \Rightarrow \begin{cases} && r_f(q_m, m) r_f(k_n, n) &=& r_g(q_m, k_n, m - n) \\ \\ && \theta_f(q_m, m) - \theta_f(k_n, n) &=& \theta_g(q_m, k_n, m - n) \end{cases} $$

代入 $n=m$，得到

$$ \begin{cases} && r_f(q_m, m) r_f(k_m, m) &=& r_g(q_m, k_m, 0) &=& r_f(q_m, 0) r_f(k_m, 0) \\ && \theta_f(q_m, m) - \theta_f(k_m, m) &=& \theta_g(q_m, k_m, 0) &=& \theta_f(q_m, 0) - \theta_f(k_m, 0) \end{cases} $$

我们不妨认为位置 0 处的 $q, k$ 参数值与位置无关。故作**假设2. $f(x, 0) = x$**. 由此 $r_f(x, 0) = ||x||, \theta_f(x, 0) = \theta(x)$，其中 $||x||$ 为 $x$ 的模长，$\theta(x)$ 为 $x$ 的幅角。

所以有

$$ \begin{cases} && r_f(q_m, m) r_f(k_m, m) &=& ||q_m|| \cdot ||k_m|| \\ && \theta_f(q_m, m) - \theta_f(k_m, m) &=& \theta(q_m) - \theta(k_m) \end{cases} $$

根据第一个式子，我们不妨作**假设3. $r_f(x, m) = ||x||$，即位置编码不改变向量的模长**。

根据第二个式子，我们有 $$ \theta_f(q_m, m) - \theta(q_m) = \theta_f(k_m, m) - \theta(k_m) $$

这表示 $ \theta_f(x, m) - \theta(x) $ 与 $x$ 无关，只与位置 $m$ 有关。我们不妨设 $ \theta_f(x, m) - \theta(x) = \phi(m) $，即 $$ \theta_f(x, m) = \theta(x) + \phi(m) $$

代入 $n = m - 1$，得到

$$ \begin{aligned} && \theta_f(q_m, m) - \theta_f(k_{m-1}, m-1) &= \theta_g(q_m, k_{m-1}, 1) \\ \\ \Rightarrow && \phi(m) - \phi(m-1) &= \theta_g(q_m, k_{m-1}, 1) + \theta(k_{m-1}) - \theta(q_m) \end{aligned} $$

因为 $\phi(m)$ 的值只与 $m$ 有关，所以上式的右侧应该是一个只与 $m$ 有关的值，然而式子中并不包含参数 $m$，所以上式右侧应为一个常数，我们设为 $\theta$. 最终得到 $$ \phi(m) = m\theta $$

## 2. RoPE 位置编码定义

由第一节的推导，我们得到 2 维情况下复数形式的位置编码

$$ \begin{aligned} f(q_m, m) &= r_f(q_m, m) e^{i\theta_f(q_m, m)} \\ &= ||q_m||e^{i(\theta(q_m) + m\theta)} \\ &= q_m e^{im\theta} \end{aligned} $$

根据复数的几何意义，上式对应着向量的旋转，可以写为

$$ f(q_m, m) = \begin{bmatrix} \cos{m\theta} & -\sin{m\theta} \\ \sin{m\theta} & \cos{m\theta} \end{bmatrix} \begin{bmatrix} {q_m}_0 \\ {q_m}_1 \end{bmatrix} $$

由于内积的线性叠加性（这里可能需要一个高维旋转矩阵的推导），对于任意偶数维的 $q, k$，我们可以将旋转表示成多个二维形式的拼接，即

$$ f(q_m, m) = R_m q_m  = \begin{bmatrix} R(m\theta_0) & 0 & ... & 0 \\ 0 & R(m\theta_1) & ... & 0 \\ ... & ... & ... & ... \\ 0 & 0 & ... & R(m\theta_{d/2-1}) \end{bmatrix} \begin{bmatrix} {q_m}_0 \\ {q_m}_1 \\ ... \\ {q_m}_{d-1} \end{bmatrix} $$

其中 $R(\theta) = \begin{bmatrix} \cos(\theta) & -\sin(\theta) \\ \sin(\theta) & \cos(\theta) \end{bmatrix}$.

沿用 Sinusoidal 位置编码的频率设计，选择 $\theta_i = 10000^{-\frac{2i}{d}}$.

### 相对位置关系

容易发现，用经过 RoPE 变换后的 $q, k$ 做注意力计算，注意力分数会自动包含相对位置信息 
$$ (R_m q_m)^T(R_n k_n) = q_m^T R_m^T R_n k = q^T R_{n-m} k $$

### 实现方式

由于 $R_m$ 是一个稀疏矩阵，直接使用矩阵乘法计算效率太低，所以一般通过以下方式实现：

$$ R_m q_m = \begin{bmatrix} {q_m}_0 \\ {q_m}_1 \\ {q_m}_2 \\ {q_m}_3 \\ ... \\ {q_m}_{d-2} \\ {q_m}_{d-1} \end{bmatrix} \otimes \begin{bmatrix} \cos{m\theta_0} \\ \cos{m\theta_0} \\ \cos{m\theta_1} \\ \cos{m\theta_1} \\ ... \\ \cos{m\theta_{d/2-1}} \\ \cos{m\theta_{d/2-1}} \end{bmatrix} + \begin{bmatrix} -{q_m}_1 \\ {q_m}_0 \\ -{q_m}_3 \\ {q_m}_2 \\ ... \\ -{q_m}_{d-1} \\ {q_m}_{d-2} \end{bmatrix} \otimes \begin{bmatrix} \sin{m\theta_0} \\ \sin{m\theta_0} \\ \sin{m\theta_1} \\ \sin{m\theta_1} \\ ... \\ \sin{m\theta_{d/2-1}} \\ \sin{m\theta_{d/2-1}} \end{bmatrix} $$

其中 $\otimes$ 表示逐元素乘法。

而实际实现中，多将向量分为两半，重新组合并使正负号计算分开，公式变为

$$ R_m q_m = \begin{bmatrix} {q_m}_0 \\ {q_m}_1 \\ ... \\ {q_m}_{d/2-1} \\ {q_m}_{d/2} \\ {q_m}_{d/2+1} \\ ... \\ {q_m}_{d-1} \end{bmatrix} \otimes \begin{bmatrix} \cos{m\theta_0} \\ \cos{m\theta_1} \\ ... \\ \cos{m\theta_{d/2-1}} \\ \cos{m\theta_0} \\ \cos{m\theta_1} \\ ... \\ \cos{m\theta_{d/2-1}} \end{bmatrix} + \begin{bmatrix} -{q_m}_{d/2} \\ -{q_m}_{d/2+1} \\ ... \\ -{q_m}_{d-1} \\ {q_m}_0 \\ {q_m}_1 \\ ... \\ {q_m}_{d/2-1} \end{bmatrix} \otimes \begin{bmatrix} \sin{m\theta_0} \\ \sin{m\theta_1} \\ ... \\ \sin{m\theta_{d/2-1}} \\ \sin{m\theta_0} \\ \sin{m\theta_1} \\ ... \\ \sin{m\theta_{d/2-1}} \end{bmatrix} $$