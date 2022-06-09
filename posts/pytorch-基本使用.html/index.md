# Pytorch 基本使用


## 前言

记录深度学习中使用 pytorch 的常用技巧

## 内容 

### 就地函数

在 pytorch 中，使用带有下标 `_` 表示的就地改变数据，而不需要使用变量接受改变后的值，比如 `fill_` 函数。

```python
x = torch.Tensor(2, 3)
x.fill_(5)
# tensor([[5., 5., 5.],
#        [5., 5., 5.]])
```

### numypy 转 Tensor

numpy 转 Tensor后，Tensor 的类型为 DoubleTensor，而不是默认的 FloatTensor，这对应于 numpy 的 float64。

### torch.Tensor 和 torch.tensor 区别

Tensor 是类，而 tenosr 是函数，Tensor 返回的是 FloatTensor，而 tensor 则根据提供的数据返回的 LongTensor、 FloatTensor 和 DoubleTensor 等类型。

### Tensor插入维度

Tensor 插入一个维度为 1 的常用方法为 unsqueeze(dim=dim)，对应的减少维度方法为 squeeze(dim=dim)。

也可以像 Numpy 一样，使用 None 插入的一个维度为 1 新维度

```python
>>> n = torch.rand(3, 100, 100)
>>> n[:, None].shape
(3, 1, 100, 100)

>>> n.unsqueeze(1).shape
(3, 1, 100, 100)
```

其中 None 所在的维度表示所要插入维度的位置。默认是从头开始表示所要插入的维度，可以使用 `...` 表示表示最为一个维度相对的位置进行插入

```python
n[..., None] 等价于 n.unsqueeze(dim=-1)
n[..., None, :] 等价于 n.unsqueeze(dim=-2)
```

[Indexing a tensor with None in PyTorch](https://stackoverflow.com/questions/69797614/indexing-a-tensor-with-none-in-pytorch)


