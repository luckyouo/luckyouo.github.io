# Pytorch 常用api


## 前言

`Pytorch` 是研究生阶段深度学习常用的工具。在学习深度学习的过程中，记录一下使用 `Pytorch` 经常遇到 `api`。

## 常用 API

### Tensor.repeat(*sizes) -> Tensor

沿指定维度重复张量 `sizes` 次数，可以同时指定多个维度，例如

```python
>>> x = torch.tensor([1, 2, 3])
>>> x.repeat(4, 2)
tensor([[ 1,  2,  3,  1,  2,  3],
        [ 1,  2,  3,  1,  2,  3],
        [ 1,  2,  3,  1,  2,  3],
        [ 1,  2,  3,  1,  2,  3]])
>>> x.repeat(4, 2, 1).size()
torch.Size([4, 2, 3])
```

上述表达式为在第 `0` 维度重复四次，在第 `1` 维度重复两次

### torch.permute(input, dims) -> Tensor

按所给顺序交换张量各个维度

```python
>>> x = torch.randn(2, 3, 5)
>>> x.size()
torch.Size([2, 3, 5])
>>> torch.permute(x, (2, 0, 1)).size()
torch.Size([5, 2, 3])
```

默认顺序从 `0` 开始，上述表达式为各个维度向右移动一位。

### torch.empty(size...)→ Tensor

安装所给的维度，初始化一个张量，但张量内数据未初始化（随机）

```python
>>> torch.empty((2,3), dtype=torch.int64)
tensor([[ 9.4064e+13,  2.8000e+01,  9.3493e+13],
        [ 7.5751e+18,  7.1428e+18,  7.5955e+18]])
```

