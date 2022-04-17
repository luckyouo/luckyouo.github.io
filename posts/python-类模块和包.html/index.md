# Python 类、模块和包


## 前言

为了更好的了解 python `import` 的功能，需要知道 python 中类、模块和包的概念和区别。

## 定义

### 类

python 作为面向对象的语言，提供了使用 `class` 关键字来定义一个类，并且支持面向对象编程（OOP）的所有标准特性：类的继承机制支持多个基类、派生的类能覆盖基类的方法、类的方法能调用基类中的同名方法。对象可包含任意数量和类型的数据。

### 模块

模块就是一个 python 文件，以 `.py` 结尾，包含了 Python 对象定义和Python语句。

### 包

python 中的包是一个文件夹，该文件夹下包含若干个模块和文件夹，且该文件夹下还必须包含一个 `__init__.py` 文件，若子文件夹下仍然含有 `__init__.py` 文件，则该文件夹为对应包的子包。

## 基本使用

### 导入类

因为类是在模块当中的，需要从模块中导入类

```python
from modle import classname
```

### 导入模块

导入整个模块：

```python
import model
```

导入模块中的所有类

```python
from model import *
```

导入某个包内的模块

```python
from package import module

# 或者
import package.module_a
```

### 常见问题

python 导包会从 `PYTHONPATH` 环境路径中逐级查找，如果没有找到则抛出异常，第一个是执行文件所在的路径，即会先从当前文件下进行搜索。常看导包的环境路径

```python
import sys
print(sys.path)
```


