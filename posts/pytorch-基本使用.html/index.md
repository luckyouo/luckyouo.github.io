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

