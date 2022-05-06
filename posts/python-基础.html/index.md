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

python3 中 `/` 除法，两个操作数会转换为 float 类型，结果而已为 float 类型，如果需保持整数类型是，需要使用 `//` 运算符。

```python
a = 10

# 死循环
while a > 10:
    a /= 10
```

#### 四舍五入

使用 round 方法对结果四舍五入

```python
print(round(10 / 3))
# 3
```

#### 取整数

舍去小数部分，只留整数部分

```python
print(int (-7 / 6))
# 0
pirnt(round(-7 / 6))
# -1
```

#### 向上取整

使用 math 库中的 ceil 方法，对结果向上取整

```python
import math
print(math.ceil(10 / 3))
# 4
```

#### 取整数和小数部分

使用 math 库中的 modf 方法，取结果的整数和小数部分，结果为二元组。

```python
import math
print(math.modf(10 / 3))
# (0.3333333333333335, 3.0)
```

参考： [Python除法：四舍五入，地板除，取整，取小数](http://www.juzicode.com/python-note-divide/) 

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


