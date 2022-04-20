# Dataset 、 Sampler 和 Dataloader 三者的关系


## 前言

深度学习导入数据常用的两个工具类就是 Dataset 和 Dataloader，其中 Dataset 是 Dataloder 实例化所需要的参数。

## 三者关系

### Dataset

Dataset 是用来装数据集的抽象类，因此必须通过继承它来实例化对象。所有 Dataset 的子类都必须重写 `__getitem__()` 方法，可选择重写 `__len__()` 方法，但 Sampler 和 Dataloader 都需要使用该方法来获取数据集的大小，所以该方法一般都会进行重写。

### Sampler

Sampler 是用来对 Dataset 对象进行抽样的，生成一系列的 index，并按照 batch 进行合并为 indices 返回。pytoch 内部实现的 Sampler 有如下几种：

- SequentialSampler：顺序采样
- RandomSampler：随机采样
- SubsetRandomSampler： 子集合随机采用
- BatchSampler：批量采样
- DistributedSampler：按某概率分布进行采样

Sampler 也会作为参数实例化 Dataloader，实例化时，会传入两个参数，`sampler` 和 `batch_sampler` 。

- 如果你自定义了`batch_sampler`,那么这些参数都必须使用默认值：`batch_size`, `shuffle`,`sampler`,`drop_last`.

- 如果你自定义了`sampler`，那么`shuffle`需要设置为`False`

- 如果`sample` 和`batch_sampler`都为`None`,那么`batch_sampler`使用Pytorch已经实现好的`BatchSampler`,而`sampler`分两种情况

  分两种情况：

  - 若`shuffle=True`,则`sampler=RandomSampler(dataset)`
  - 若`shuffle=False`,则`sampler=SequentialSampler(dataset)`

### Dataloader

 Dataloader 实例方法如下:

`torch.utils.data.DataLoader`(*dataset*, *batch_size=1*, *shuffle=False*, *sampler=None*, *batch_sampler=None*, *num_workers=0*, *collate_fn=None*, *pin_memory=False*, *drop_last=False*, *timeout=0*, *worker_init_fn=None*, *multiprocessing_context=None*, *generator=None*, ***, *prefetch_factor=2*, *persistent_workers=False*)，

- num_workers: 多进程的数量，正整数，使用多进程加载数据。当在 windows 下使用时，需要设置为 0，否则会抛出异常（历史遗留bug，也可以通过其他设置修正）
- collate_fn：整合函数，为 None 时，则默认为 `default_collate()` 。

```python
default_collate([(0, 1), (2, 3)])
# [tensor([0, 2]), tensor([1, 3])]
default_collate([[0, 1], [2, 3]])
# [tensor([0, 2]), tensor([1, 3])]
```

- pin_memory：在返回数据之前，会将数据拷贝至 cuda 上
- drop_last：将最后一个批量数据丢弃

Dataloader 对象通过 foreach 循环自动调用 `__next__()` ，`__next__()` 函数会自动调用 Sampler 的 `__iter__()` 的方法， `__iter__()` 方法会自动调用 Dataset 的 `__getitem__()` 来获取对象（python 的迭代器实现的功能 [迭代器与生成器](https://python3-cookbook.readthedocs.io/zh_CN/latest/chapters/p04_iterators_and_generators.html) ）。

## 参考资料

[一文弄懂Pytorch的DataLoader, DataSet, Sampler之间的关系](https://www.cnblogs.com/marsggbo/p/11308889.html)

[Pytorch doc](https://pytorch.org/docs/stable/data.html#torch.utils.data.DataLoader)

