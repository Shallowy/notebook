# Lab 1 Pytorch101

## 1.1 Tensor基础 Basics

### 创建Tensor

```python
# 从list创建tensor
a = torch.tensor([1, 2, 3])
b = torch.tensor([[1, 2, 3], [4, 5, 6]])

# 用构造器创建tensor
c = torch.zeros(2, 3)     # 2×3的全0矩阵
d = torch.ones(1, 2)      # 1×2的全1矩阵
e = torch.eye(3)          # 3×3的单位矩阵
f = torch.full((2, 3), 7) # 2×3的全7矩阵
g = torch.rand(4, 5)      # 4×5的随机[0, 1)矩阵
h = torch.arange(0, 10, 2) # 0（可省略，默认0）到10（不包含10）的等差数列，步长为2
```

### Tensor属性

- 直接访问 `a[0]` 返回Pytorch的Tensor对象，可以通过 `a[0].item()` 获取Python的标量值

```python
# tensor值
print(a) # tensor([1, 2, 3])
print(b) # tensor([[1, 2, 3],
         #         [4, 5, 6]])
print(a[0])    # tensor(1)
print(b[0])    # tensor([1, 2, 3])
print(b[0, 1]) # tensor(2)

# tensor类型
print(type(a))           # <class 'torch.Tensor'>
print(type(b))           # <class 'torch.Tensor'>
print(type(a[0]))        # <class 'torch.Tensor'>
print(type(a[0].item())) # <class 'int'>

# tensor维度
print(a.dim()) # 1
print(b.dim()) # 2

# tensor形状
print(a.shape) # torch.Size([3])
print(b.shape) # torch.Size([2, 3])

# tensor元素数量（维度的乘积）
print(a.numel()) # 3
print(b.numel()) # 6
```

### 数据类型

- Pytorch提供了多种数据类型，创建Tensor时可以指定数据类型，若不指定则Pytorch会自动推断
- 每个Tensor都有一个数据类型，可以通过 `dtype` 属性访问

```python
# 自动推断数据类型
x0 = torch.tensor([1, 2])   # 整数列表
x1 = torch.tensor([1., 2.]) # 浮点数列表
x2 = torch.tensor([1., 2])  # 混合列表
print(x0.dtype) # torch.int64
print(x1.dtype) # torch.float32
print(x2.dtype) # torch.float32

# 指定数据类型
y0 = torch.tensor([1, 2], dtype=torch.float32) # 32-bit float
y1 = torch.ones(1, 2, dtype=torch.uint8)       # 8-bit (unsigned) integer

# 数据类型转换：指定数据类型
z0 = torch.eye(3, dtype=torch.int64)
z1 = z0.float()           # 转换为32-bit float
z2 = z0.double()          # 转换为64-bit float
z3 = z0.to(torch.float32) # 转换为32-bit float（与z1等价）

# 数据类型转换：指定与某Tensor相同的数据类型
z0 = torch.eye(3, dtype=torch.float64)
z1 = torch.zeros_like(z0)    # 与z0形状、数据类型相同的全0矩阵
z2 = z0.new_zeros(4, 5)      # 与z0数据类型相同的4×5全0矩阵
z3 = torch.ones(6, 7).to(z0) # 与z0数据类型相同的6×7全1矩阵
```

## 1.2 Tensor索引 Indexing

### 切片索引
```python
# 一维Tensor
a = torch.tensor([0, 11, 22, 33, 44, 55, 66])
print(a[2:5])   # 索引2到4的元素
print(a[2:])    # 索引2到最后的元素
print(a[:5])    # 索引0到4的元素
print(a[:])     # 所有元素
print(a[1:5:2]) # 索引1到4的元素，步长为2
print(a[:-1])   # 索引0到倒数第二个元素
print(a[-4::2]) # 索引倒数第四个到最后的元素，步长为2

# 二维Tensor
b = torch.tensor([[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12]])
print(b[1, :])     # tensor([5, 6, 7, 8])
print(b[1])        # tensor([5, 6, 7, 8])（":"可忽略）
print(b[:, 1])     # tensor([2, 6, 10])
print(b[:2, -3:])  # tensor([[2, 3, 4],
                   #         [6, 7, 8]])
print(b[::2, 1:3]) # tensor([[2, 3],
                   #         [10, 11]])

row1 = b[1, :]
row2 = b[1:2, :]
print(row1, row1.shape) # tensor([5, 6, 7, 8]) torch.Size([4])
print(row2, row2.shape) # tensor([[5, 6, 7, 8]]) torch.Size([1, 4])
col1 = b[:, 1]
col2 = b[:, 1:2]
print(col1, col1.shape) # tensor([ 2,  6, 10]) torch.Size([3])
print(col2, col2.shape) # tensor([[ 2],
                        #         [ 6],
                        #         [10]]) torch.Size([3, 1])
```

#### 切片索引的引用问题

- 切片索引返回的是原Tensor的视图（view），修改切片索引的值会改变原Tensor的值
- 使用 `clone()` 可以获得切片索引的副本

```python
a = torch.tensor([[1, 2, 3, 4], [5, 6, 7, 8]])
b = a[0, 1:]
c = a[0, 1:].clone()

a[0, 1] = 20
b[1] = 30
c[2] = 40

print(a) # tensor([[ 1, 20, 30,  4],
        #         [ 5,  6,  7,  8]])
print(b) # tensor([20, 30,  4])
print(c) # tensor([ 2,  3, 40])
```

#### 切片索引赋值

- 切片索引赋值语句的右侧可以是标量、列表、Tensor等

```python
a = torch.zeros(2, 4, dtype=torch.int64)
a[:, :2] = 1
a[:, 2:] = torch.tensor([[2, 3], [4, 5]])
print(a) # tensor([[1, 1, 2, 3],
         #         [1, 1, 4, 5]])
```

### 索引数组

```python
a = torch.tensor([[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12]])
idx = [0, 0, 2, 1, 1]
print(a[idx]) # tensor([[1, 2, 3, 4],
              #         [1, 2, 3, 4],
              #         [9, 10, 11, 12],
              #         [5, 6, 7, 8],
              #         [5, 6, 7, 8]])
idx = torch.tensor([3, 2, 1, 0])
print(a[:, idx]) # tensor([[ 4,  3,  2,  1],
                 #         [ 8,  7,  6,  5],
                 #         [12, 11, 10,  9]])

b = torch.tensor([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
idx = [0, 1, 2]
b[idx, idx] = torch.tensor([11, 22, 33])
print(b) # tensor([[11,  2,  3],
         #         [ 4, 22,  6],
         #         [ 7,  8, 33]])

c = torch.tensor([[1, 2, 3], [4, 5, 6], [7, 8, 9], [10, 11, 12]])
idx0 = torch.arange(a.shape[0]) # [0, 1, 2, 3]
idx1 = torch.tensor([1, 2, 1, 0])
print(c[idx0, idx1]) # tensor([ 2,  6,  8, 10])
c[idx0, idx1] = 0
print(c) # tensor([[ 1,  0,  3],
         #         [ 4,  5,  0],
         #         [ 7,  0,  9],
         #         [ 0, 11, 12]])
```

### 布尔索引

```python
a = torch.tensor([[1,2], [3, 4], [5, 6]])
mask = (a > 3)
print(mask)    # tensor([[False, False],
               #         [False,  True],
               #         [ True,  True]])
print(a[mask]) # tensor([4, 5, 6])
num = torch.sum(a <= 3) # 3
a[a <= 3] = 0
print(a)                # tensor([[0, 0],
                        #         [0, 4],
                        #         [5, 6]])
```

## 1.3 改变Tensor形状 Reshaping
### view

- `view()` 方法返回一个新的Tensor，与原Tensor元素数量相同，但形状不同
- `view()` 方法返回的新Tensor与原Tensor共享数据存储空间，修改一个会影响另一个

```python
x0 = torch.tensor([[1, 2, 3, 4], [5, 6, 7, 8]])
x1 = x0.view(8)       # tensor([1, 2, 3, 4, 5, 6, 7, 8])
x2 = x0.view(1, 8)    # tensor([[1, 2, 3, 4, 5, 6, 7, 8]])
x3 = x0.view(8, 1)    # tensor([[1],
                      #         [2],
                      #         [3],
                      #         [4],
                      #         [5],
                      #         [6],
                      #         [7],
                      #         [8]])
x4 = x0.view(2, 2, 2) # tensor([[[1, 2],
                      #          [3, 4]],

                      #         [[5, 6],
                      #          [7, 8]]])
x5 = x0.view(-1, 4)   # tensor([[1, 2, 3, 4],
                      #         [5, 6, 7, 8]])
```

### 交换维度

- `t()` 方法可以交换二维Tensor的维度
- `transpose()` 方法可以交换多维Tensor的两个维度
- `permute()` 方法可以交换多维Tensor的多个维度

```python
x = torch.tensor([[1, 2, 3], [4, 5, 6]]) # tensor([[1, 2, 3],
                                         #         [4, 5, 6]])
y = x.t()      # tensor([[1, 4],
               #         [2, 5],
               #         [3, 6]])
z = torch.t(x) # tensor([[1, 4],
               #         [2, 5],
               #         [3, 6]])

x0 = torch.tensor([
     [[1,   2,  3,  4],
      [5,   6,  7,  8],
      [9,  10, 11, 12]],

     [[13, 14, 15, 16],
      [17, 18, 19, 20],
      [21, 22, 23, 24]]])
x1 = x0.transpose(1, 2)   # tensor([[[ 1,  5,  9],
                          #          [ 2,  6, 10],
                          #          [ 3,  7, 11],
                          #          [ 4,  8, 12]],
                          # 
                          #         [[13, 17, 21],
                          #          [14, 18, 22],
                          #          [15, 19, 23],
                          #          [16, 20, 24]]])
x2 = x0.permute(1, 2, 0)  # tensor([[[ 1, 13],
                          #          [ 2, 14],
                          #          [ 3, 15],
                          #          [ 4, 16]],
                          # 
                          #         [[ 5, 17],
                          #          [ 6, 18],
                          #          [ 7, 19],
                          #          [ 8, 20]],
                          # 
                          #         [[ 9, 21],
                          #          [10, 22],
                          #          [11, 23],
                          #          [12, 24]]])
```


### 连续存储

- `transpose()` 不会改变Tensor的底层存储顺序，而只是交换了维度的访问方式，但 `view()` 只能用于连续存储的Tensor，所以需要先调用 **`contiguous()`** 让数据在内存中连续存储，或者使用 **`reshape()`** 方法代替 `view()`.

```python
x0 = torch.randn(2, 3, 4)
x1 = x0.transpose(1, 2).view(8, 3)              # Error
x2 = x0.transpose(1, 2).contiguous().view(8, 3) # OK
x3 = x0.transpose(1, 2).reshape(8, 3)           # OK
```

## 1.4 Tensor运算 Operations

### 按元素运算

```python
x = torch.tensor([[1, 2, 3, 4]], dtype=torch.float32)
y = torch.tensor([[5, 6, 7, 8]], dtype=torch.float32)
print(x + y, torch.add(x, y), x.add(y))  # tensor([[ 6.,  8., 10., 12.]])
print(x - y, torch.sub(x, y), x.sub(y))  # tensor([[-4., -4., -4., -4.]])
print(x * y, torch.mul(x, y), x.mul(y))  # tensor([[ 5., 12., 21., 32.]])
print(x / y, torch.div(x, y), x.div(y))  # tensor([[0.2000, 0.3333, 0.4286, 0.5000]])
print(x ** y, torch.pow(x, y), x.pow(y)) # tensor([[1.0000e+00, 6.4000e+01, 2.1870e+03, 6.5536e+04]])

print(torch.sqrt(x), x.sqrt()) # tensor([[1.0000, 1.4142, 1.7321, 2.0000]])
print(torch.sin(x), x.sin())   # tensor([[ 0.8415,  0.9093,  0.1411, -0.7568]])
```

### 缩减操作

- 默认情况下，缩减操作会缩减Tensor的所有维度，返回一个标量；可以通过 `dim` 参数指定沿着哪个维度进行缩减，此时仅该维度会被缩减

```python
x = torch.tensor([[1, 2, 3],
                  [4, 5, 6]], dtype=torch.float32)
print(torch.sum(x), x.sum())             # tensor(21.)
print(torch.sum(x, dim=0), x.sum(dim=0)) # tensor([5., 7., 9.])
print(torch.sum(x, dim=1), x.sum(dim=1)) # tensor([ 6., 15.])
```

### 矩阵运算

- `torch.dot()` 计算两个向量的内积
- `torch.mm()` 计算两个矩阵的乘积
- `torch.mv()` 计算矩阵和向量的乘积
- `torch.matmul()` 计算两个Tensor的乘积，根据输入的维度自动选择是矩阵乘积还是按元素乘积
- `torch.bmm()` 计算两个batch的矩阵乘积

```python
v = torch.tensor([9,10], dtype=torch.float32)
w = torch.tensor([11, 12], dtype=torch.float32)
print(torch.dot(v, w), v.dot(w)) # tensor(219.)

x = torch.tensor([[1,2],[3,4]], dtype=torch.float32)
y = torch.tensor([[5,6],[7,8]], dtype=torch.float32)
print(torch.mm(x, y), x.mm(y)) # tensor([[19., 22.],
                               #         [43., 50.]])

print(torch.mv(x, v), x.mv(v)) # tensor([29., 67.])

print(torch.matmul(x, v), x.matmul(v)) # tensor([29., 67.])
```

### 向量化

- Pytorch的运算是向量化的，即运算函数会自动处理Tensor的每个元素，无需编写循环
- 向量化运算通常比循环运算更快，所以应尽量使用向量化运算

### 广播机制

- Pytorch支持广播（broadcasting）机制，即不同形状的Tensor进行运算时会自动扩展为相同形状
- 广播机制的规则如下：
    1. 若两个Tensor的维度不同，通过在维度较小的形状前面补1，使得两个Tensor的维度相同
    2. 我们称两个Tensor在某个维度是相容的，如果它们的形状在该维度上相同，或者其中一个Tensor在该维度上的长度为1
    3. 如果两个Tensor在所有维度上都是相容的，它们就能使用广播
    4. 在任何一个维度上，如果一个Tensor的长度为1，另一个Tensor长度大于1，那么在该维度上，就好像是对第一个Tensor进行了复制
    5. 广播之后，每个Tensor的形状是输入Tensor形状的逐元素最大值

```python
x = torch.tensor([[1, 2, 3], [4, 5, 6], [7, 8, 9], [10, 11, 12]])
v = torch.tensor([1, 0, 1])
y = x + v # 等价于 x + v.repeat(4, 1)
print(y)  # tensor([[ 2,  2,  4],
          #         [ 5,  5,  7],
          #         [ 8,  8, 10],
          #         [11, 11, 13]])

```

### GPU加速

```python
x0 = torch.tensor([[1, 2], [3, 4]], dtype=torch.float32)
print(x0.device) # cpu

x1 = x0.to('cuda') # cuda:0
x2 = x0.cuda()     # cuda:0
x3 = x1.to('cpu')  # cpu
x4 = x1.cpu()      # cpu

y = torch.tensor([[1, 2, 3], [4, 5, 6]], dtype=torch.float64, device='cuda') # cuda:0
x5 = x0.to(y) # cuda:0
```