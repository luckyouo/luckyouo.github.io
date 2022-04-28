# Python 基础


## 前言

记录一些 python 的基础使用

## 内容

### 两个列表相加

python 中两个列表相加时，是将两个列表合并为一个列表，合并顺序与相加顺序一致。这与 pytorch 的向量相加不一样，pytorch 两个向量相加则是对应位置值相加（按照广播原则进行）。

```python
arr = [1, 2]
out = [3] + arr
print(out)
## [3, 1, 2]
```

### 除法运算

python 中 `/` 除法会结果会自动转换为 float 类型，如果需保持整数类型是，需要使用 `//` 运算符。

```python
a = 10

# 死循环
while a > 10:
    a /= 10
```

### nolocal 关键字

函数嵌套定义时，内部函数变量默认为局部变量，外部函数无法访问，即内部函数调用完成后销毁。若需要将内部变量提供给外部函数时，可以使用关键字 `nolocal` 关键字修饰

```python
def myfunc1():
  x = "John"
  def myfunc2():
    nonlocal x
    x = "hello"
  myfunc2()
  return x

print(myfunc1())
```


