# Python 基础


## 前言

记录一些 python 的基础使用

## 内容

### 常用的魔法方法

- `__len__()` :  当使用 `len` 函数时，会自动调用该方法，若类没有使用该方法，则抛出异常
- `__getitem__()` : 当使用切片、索引、for-in 循环时候，会自动调用该方法，若类实现了该方法，则可以使用索引、切片对访问类中的元素
- `__iter__()` : 使用 for-in 循环会自动调用该方法，若类实现了该方法，则表明该类是可以迭代的
- `__abs__()` : 使用 `abs` 函数会自动调用该方法
- `__add__()` : 对类对象使用 ”+“ 运算符，会自动调用该方法，返回一个新的对象
- `__mul__()` : 对类对象使用 "*" 运算符，会自动调用该方法，返回一个新的对象
- `__repr__()` : 在终端直接输出对象时，调用该方法
- `__str__()` : 使用 print 打印对象时，调用该方法。如果没有，则使用 `__repr__ ` 方法进行替代
- `__bool__()` ：使用 `bool` 函数时，会自动调用该方法，使用 if 、while 语句时，也会自动调用该方法，因为判断对象是否为真会调用 `bool` 函数。如果 `__bool__` 方法不存在时，则调用 `__len__ ` 方法进行判断，长度为 0 为假。**默认对象都是为真** 。
- `__iadd__()` : 自增运算，当使用 += 运算符操作对象时，会自动调用该方法。若没有该方法，则会后退调用 `__add__` 方法。
- `__call__()` : 实现了该方法的类的对象，它的实例可以作为函数进行调用。

### 自增赋值

使用 += 等自增运算符时，表示原地自增。而使用 + 运算符，则生成了新的对象，并赋值。比如 a += b，则在 a 的基础上直接增加，不会生成新的变量。而 a = a + b ，则表示 a + b 生成新的对象，并将该对象的引用赋值给 a 。一般可变序列都实现了 `__iadd__` 等自增运算符，不可变序列则不支持该方法。 

### 序列的拼接

python 中两个列表相加时，是将两个列表合并为一个列表，合并顺序与相加顺序一致。这与 pytorch 的向量相加不一样，pytorch 两个向量相加则是对应位置值相加（按照广播原则进行）。

```python
arr = [1, 2]
out = [3] + arr
print(out)
## [3, 1, 2]
```

使用 $*$ 运算符进行序列拼接时，若元素为引用对象的引用的话，则拼接的多个序列都指向一个对象。若发生修改，则都进行修改。

```python
weird_board = [['_'] * 3] * 3 
weird_board[1][2] = 'O'
# [['_', '_', 'O'], ['_', '_', 'O'], ['_', '_', 'O']]
```

此时，需要使用列表推导式进行生成

```python
board = [['_'] * 3 for i in range(3)] 
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

### 列表推导式的作用域

在 python2.* 版本中，列表推导式的变量会影响外部变量，但在 python3 中，列表推导式为局部作用域，不会影响外部变量

```python
x = 2
dummy = [x for x in 'abc']
print(x)
# 2
```

### 列表推导式的嵌套

列表推导式的 for 循环嵌套与书写顺序一致，生成相应的笛卡尔积结果

```python
product = [(x, y) for x in range(3) for y in range(3,5)]
# (0, 3) (0, 4) ...
```

### 切片赋值

如果把切片放在赋值语句的左边,或把它作为 del 操作的对象,我们就可以对序列进行嫁接、切除或就地修改操作。如果赋值的对象是一个切片，那么赋值语句的右侧必须是可以迭代的对象。**赋值的长度不需要一致**

### 元组自增

元组是不支持自增运算的，但是元组内部的若含有可变对象时，则可以进行自增，但是自增之后因为于元组内数据发生变化，则会抛出异常。（说明自增运算不是原子操作，普通 + 则直接抛出异常，且数据不会发生变更）

```python
t = (1, 2, [30, 40])
t[2] += [50, 60] # 抛出异常，但 t 已经发生变化
print(t) # t = (1, 2, [30, 40, 50, 60])
```

所以，元组内不应该放置可变元素

### 序列排序

`list.sort()` 会对序列原地排序，所以返回结果为 None，而内置的函数 `sorted()` 则会返回排序好的序列，该方法需要序列可以进行迭代，且两个方法都有两个关键字：

1. reverse：为 True 时，进行逆序排列。
2. key： 一个只有一个参数的函数，该函数会应用到所有元素上，并使用结果进行排序，默认恒等函数。

### 装饰器

python 的装饰期在模块加载之后就立即执行。装饰器其实就是一个高阶函数，是一个语法糖。参数化的装饰器本质为两层嵌套函数，外层相当于装饰器工厂，根据参数返回需要的装饰器。

常用的装饰器：

1. `functools.lru_cache()`

该装饰器实现了备忘的功能，并使用最近最少用策略管理缓存。 `cache` 装饰器不清理缓存，更快。这些在需要递归的函数中使用。**可以在算法中需要使用记忆花搜索时使用**

```python
import functools

@functools.lru_cache()
@clock # 自己实现的查看函数执行时间的装饰器
def fib(n):
    if n  < 2:
        return n
    return fib(n-1) + fib(n-2)
```

必须像常规函数那样调用 lru_cache。这一行中有一对括号: `@functools.lru_cache()`。这么做的原因是，`lru_cache` 可以接受配置参数。使用该装饰器之后，这样每个 n 都只会执行一次。

该装饰器的签名 `functools.lru_cache(maxsize=128, typed=False)` 。maxsize 参数指定存储多少个调用的结果。缓存满了之后,旧的结果会被扔掉,腾出空间。为了得到最佳性能，maxsize 应该设为 2 的幂。typed 参数如果设为 True，把不同参数类型得到的结果分开保存，即把通常认为相等的浮点数和整数参数(如 1 和 1.0)区分开。

因为 `lru_cache` 使用了哈希散列存储，所以被装饰器的函数中的的参数都必须是可以散列的。

2. `functools.singledispatch`

由于 python 函数不支持重载，所以如果需要编写同名方法的函数，只能使用 `if-else` 调用不同的函数执行，于是则各个函数之间耦合过于紧密。python3.4 增加了 `functools.singledispatch` 装饰器用于解决该问题。

```python
from functools import singledispatch
from collections import abc
import numbers
import html

@singledispatch ➊
def htmlize(obj):
    content = html.escape(repr(obj))
    return '<pre>{}</pre>'.format(content)

@htmlize.register(str) ➋
def _(text):
    content = html.escape(text).replace('\n', '<br>\n')
    return '<p>{0}</p>'.format(content)

@htmlize.register(numbers.Integral) ➍
def _(n):
	return '<pre>{0} (0x{0:x})</pre>'.format(n)

@htmlize.register(tuple) ➎
@htmlize.register(abc.MutableSequence)
def _(seq):
    inner = '</li>\n<li>'.join(htmlize(item) for item in seq)
	return '<ul>\n<li>' + inner + '</li>\n</ul>'
```

singledispatch 创建一个自定义的 htmlize.register 装饰器，把多个函数绑在一起组成一个泛函数。



### 作用域

如果在函数体的内部定义了一个与全部变量同名的局部变量，则 python 会优先将该变量是作为局部变量，如果在赋值之前使用了该变量，则会抛出为引用的异常。

```python
b = 3
def f(a):
    print(a)
    print(b) # b 为局部变量，为初始化使用抛出异常
    b = 6
```

如果要在函数中使用全局变量，则应该使用 `global` 关键字。

### 闭包

闭包是一种函数，它会保留定义函数时存在的自由变量的绑定，这样调用函数时，虽然定义作用域不可用了，但是仍能使用那些绑定。当闭包内使用的是不可变量时，若发生变化，则会变成局部变量，而不是自由变量。因此，则闭包内使用不可变量时，需要使用 `nolocal` 关键字，将不可变量标记为自由变量。

```python
def make_averager():
    series = []
        def averager(new_value):
            series.append(new_value) # 自由变量
            total = sum(series)
        	return total/len(series)
    return averager
```

```python
def make_averager():
    count = 0
    total = 0
    def averager(new_value):
        count += 1	# 局部变量
        total += new_value # 局部变量
        return total / count
    return averager
```

### is 和 == 区别

is 在 python 中比较的是两个对象的 id，在 cpython 解释器中，id 即为对象的内存地址。而 == 则是调用对象的 `__eq__` 方法。这与 java 实现存在差异，在 java 中 == 比较是内存的地址。速度来说， is 直接比较内存地址，而 == 则需要调用方法，故 is 更快。

### 浅复制

序列的构造方法和切片生成的结果都是浅复制，如果里面的元素都是不可变对象时，则没有差异，如果是引用对象时，则会同时发生变化，和 java 一致。

copy 模块提供了 deepcopy 函数进行深拷贝。

### 引用传参

Python 唯一支持的参数传递模式是共享传参(call by sharing)，即引用传参数。共享传参指函数的各个形式参数获得实参中各个引用的副本。也就是说,函数内部的形参是实参的别名。

所以不应该将可变量作为默认参数进行传递。如果使用可变对象作为默认参数，则所有默认参数都将共享一个参数，会出现奇怪 bug。

### del

del 语句删除名称,而不是对象。del 命令可能会导致对象被当作垃圾回收,但是仅当删除的变量保存的是对象的最后一个引用,或者无法得到对象时。

当对象的引用数量归零后,垃圾回收程序会把对象销毁。但是,有时需要引用对象,而不让对象存在的时间超过所需时间。这经常用在缓存中。弱引用不会增加对象的引用数量。引用的目标对象称为所指对象(referent)。因此我们说,
弱引用不会妨碍所指对象被当作垃圾回收。

