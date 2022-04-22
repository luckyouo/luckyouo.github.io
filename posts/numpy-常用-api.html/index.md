# Numpy 常用 api


## 前言

记录 numpy 的常用 api 使用

## 常用 api

### numpy.stack(arrays,axis=0**,** out=None)

将 python 的 arrays 转换为 ndarray，默认是按照第一维度堆叠起来。可以使用 list 先收集向量，在按照某维度转换为 ndarray。

```python
list = [[1,2], [3,4]]
M = np.stack(list)
# array([[1, 2],[3, 4]])
M = np.stack(list, axis=-1)
# array([[1, 3],[2, 4]])
```


