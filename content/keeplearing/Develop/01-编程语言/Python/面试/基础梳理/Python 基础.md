## Python 基本数据类型
**不可变数据类型**：`int`、`float`、`bool`、`str`、`tuple`

**可变数据类型**：`list`、`set`、`dict`

> Python 中说某个类型“可变”与否，是说**对象的值（内容）能否原地改变**。  
   可变类型如 `list`、`dict` 可以**原地改**；不可变类型如 `int`、`str`、`tuple` **一旦创建就不能改，只能重新创建新对象**。
   给 `int` 类型变量重新赋值，不是修改原对象，而是**创建了一个新对象并重新绑定变量名**

## Python 列表推导式和 for 循环的区别
### 介绍
列表推导式是 Python 中一种简洁、表达力强的语法结构，用于**从一个或多个可迭代对象创建新的列表**。
它本质上是循环、条件判断和元素转换操作的单行语法糖，在大多数简单场景下比传统的 for 循环更清晰、更简短，且通常性能更高。
### 基本语法结构
```python
[ 表达式 for 变量 in 可迭代对象 if 条件 ]
```
- **表达式**：对每个元素进行的计算或变换结果（最终成为列表中的元素）
- **for 变量 in 可迭代对象**：循环部分（必须）
- **if 条件**：可选的过滤条件，只有满足条件的元素才会进入结果列表
### 区别
#### 1. 语法与表达能力
列表生成式是表达式，能够直接作为函数参数或返回值使用。
```python
res = [x * 2 for x in data if x > 0]
```
而`for`循环是语句块，必须先声明容器再逐步填充：
```python
res = []
for x in data:
    if x > 0:
        res.append(x * 2)
```
生成式更偏向声明式风格，循环更偏向过程式描述。
#### 2. 性能差异
在 CPython 中，列表生成式由解释器在 C 层面做了部分优化，通常比等价的 `for` 循环略快，尤其是在简单映射与过滤场景下更明显。但当循环体逻辑复杂时，这一优势会迅速消失。
#### 3. 可读性与可维护性
当逻辑为“一次遍历 + 简单条件 + 映射”时，列表生成式更紧凑且语义清晰。  
当存在多层嵌套、多条件分支、异常处理或副作用时，生成式会显著降低可读性，此时 `for` 循环的分步结构更有利于理解与调试。
#### 4. 副作用与工程语义
列表生成式适合纯函数式风格，不应包含日志输出、外部状态修改等副作用操作。  
`for` 循环则更适合承载过程控制逻辑与副作用。

## Python 的`is`和 `==` 有什么区别
`is`：**是否是同一个对象**
- 比较的是两个对象的**内存地址是否相同**（是否指向同一个对象）。
- 只有当两个变量引用的是同一个对象时，`is` 才返回 `True`。
`==`：**值（内容）是否相等**
- 比较的是两个对象的**值是否相等**。
- 是通过调用对象的 `__eq__()` 方法实现的。
- 即使是两个不同的对象，只要它们的内容一样，`==` 就返回 `True`。
> **`is` 只用于判断身份，如和 `None` 比较时才推荐用 `is`**

## 导入的 python 包中的 `__init__.py`文件作用
在 Python 中，包目录下的 `__init__.py` 文件承担着**包初始化入口和控制导入时的命名空间与便捷导入**两项核心职责。
### 1. 标识并初始化包
当执行 `import mypkg` 或 `from mypkg import xxx` 时，解释器首先加载并执行 `mypkg/__init__.py`。因此，该文件是包级别的“构造函数”，可用于完成只需执行一次的初始化逻辑，例如注册全局对象、加载配置或准备运行环境。
### 2. 控制导入时的命名空间与便捷导入
在 `__init__.py` 中可以编写导入语句，将子模块或子包中的常用符号提升到包的顶级命名空间，简化用户代码。
```python
# mypkg/__init__.py
from .db import connect
from .service import UserService

__all__ = ["connect", "UserService"]
```
此时调用方可以直接：
```python
from mypkg import connect, UserService
```
而无需关心 `db`、`service` 等内部模块的存在。`__all__` 进一步约束了 `from mypkg import *` 的导出范围。
## Python `__new__`和`__init__`方法区别
| 特性       | `__new__`                       | `__init__`               |
| -------- | ------------------------------- | ------------------------ |
| **作用**   | 负责创建对象（分配内存、返回实例）               | 负责初始化对象（设置属性等）           |
| **返回值**  | 必须返回一个实例（通常是 `super().__new__`） | 没有返回值（返回值会被忽略）           |
| **调用时机** | 在对象创建之前（类实例化时最先被调用）             | 在对象创建之后（`__new__` 成功返回后） |
| **常用场景** | 实现**单例模式**、元类控制等高级用途            | 给对象设置初始值是常规使用            |

## Python 深浅拷贝
**浅拷贝（Shallow Copy）**：复制最外层对象，**内部嵌套对象仍然指向原有对象**
**深拷贝（Deep Copy）**：复制对象的所有层级，**原对象与副本完全独立**
## `with open()` 介绍
**优势**：使用 `with` 语句可以**自动管理资源**，即使中途发生异常，文件也一定会被关闭！
**背后封装：**
它利用了 **上下文管理器（Context Manager）** 的机制，也就是对象实现了 `__enter__` 和 `__exit__` 方法。
等价写法：
```python
f = open("example.txt", "r")
try:
    content = f.read()
finally:
    f.close()
```
用 `with open(...)` 实际上就是 Python 帮你做了上面的 try-finally 封装！

## Python 迭代器、生成器、装饰器？metaclass、args、kwargs
### 迭代器
**定义：** 实现了 `__iter__()` 和 `__next__()` 方法的对象就是迭代器，保存的是**获取数据的方式**而不是结果，所以想用的时候就可以生成，**节省内存空间**，它是一个可以记住遍历的位置的对象。
**用途**：提供统一访问集合元素方式，节省内存
```python
it = iter([1, 2, 3])  # 得到一个迭代器
print(next(it))       # 输出 1
print(next(it))       # 输出 2
```
**特点**：
- 只能往前迭代，不能回头
- 每次取值都用 `next()`，直到抛出 `StopIteration`
### 生成器
**定义：** 生成器是特殊的迭代器，使用 `yield` 和`next`函数，有生成器表达式，生成器函数，都是为了节约内存。
**用途**：延迟计算，大量数据处理场景
```python
def my_gen():
    yield 1
    yield 2
    yield 3

for i in my_gen():
    print(i)
```
**优点**：
- 延迟计算、节省内存（一次返回一个值）
- 可中断执行，状态自动保存
### 装饰器
**定义：** 本质是一个返回函数的函数，用来**包装函数或类，增强功能**。
**用途**：在**不修改源代码**的情况下，**为函数动态增加新功能**
```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before function")
        result = func(*args, **kwargs)
        print("After function")
        return result
    return wrapper

@my_decorator
def say_hello(name):
    print(f"Hello {name}")

say_hello("Alice")
```
等价于：
```python
say_hello = my_decorator(say_hello)
```

### 元类`metaclass`
**定义：** 元类是“创建类的类”，控制类的创建过程。`type` 是最常见的元类。
**用途**：控制类的创建过程
```python
# 自定义元类
class MyMeta(type):
    def __new__(cls, name, bases, dct):
        print(f"Creating class {name}")
        return super().__new__(cls, name, bases, dct)

class MyClass(metaclass=MyMeta):
    pass
```
应用场景：
- ORM 框架（如 Django）中自动注册模型
- 自动添加方法、校验属性等
### `*args`
`*args`：**表示可变数量的位置参数**（Positional Arguments）
**用途**：可以传入任意多个参数，`args` 会以**元组形式**接收它们。
```python
def func(*args):
    print(args)

func(1, 2, 3)  # 输出: (1, 2, 3)  --> args 是一个元组
```
### `**kwargs`
`**kwargs`：**表示可变数量的关键字参数**（Keyword Arguments）
**用途**：可以传入任意多个 `key=value` 形式的参数，`kwargs` 会以**字典形式**接收它们。
```python
def func(**kwargs):
    print(kwargs)

func(name='Tom', age=18)  
# 输出: {'name': 'Tom', 'age': 18}  --> kwargs 是一个字典
```

## Python 类方法和实例方法区别
### 实例方法（Instance Method）
实例方法是最常见的方法类型。
- **定义方式：** 像普通函数一样定义，但必须将 **`self`** 作为**第一个参数**。
- **功能：** 它们操作和修改**实例**（对象）的状态。
- **如何访问：**
    - 可以通过实例对象调用。
    - 通过 `self` 参数，它们可以访问属于该实例的任何属性和方法。
```python
class Dog:
    def __init__(self, name):
        self.name = name  # 实例属性

    # 实例方法
    def bark(self):
        # 通过 self 访问实例数据 (name)
        print(f"{self.name} says Woof!")

# 调用：必须通过实例
dog1 = Dog("Fido")
dog1.bark()  # 输出: Fido says Woof!
```
### 类方法（Class Method）
**定义方式**：
```python
class MyClass:
    @classmethod
    def class_method(cls):
        print("这是类方法")
```
**特点**：
- 第一个参数是 `cls`，表示**类本身**。
- 可以通过**类或实例调用**（`MyClass.class_method()`）。
- 可以访问和修改类变量，**不能访问实例变量**。
**应用场景**：
- 工厂方法（创建类的不同实例）
- 修改/访问类级别的数据
- 实现**与实例无关的逻辑**
```python
class Dog:
    species = "Canine"  # 类变量

    def __init__(self, name):
        self.name = name

    @classmethod
    def get_species(cls):  # 类方法
        return cls.species

print(Dog.get_species())  # 输出：Canine
```

### 静态方法（@staticmethod）
```python
class Math:
    @staticmethod
    def add(a, b):
        return a + b
```
- 无 `self`、无 `cls`
- 类和实例都能调用
- 与类无关，仅组织逻辑的作用

### 类变量和实例变量区别
**类变量**：定义在类的**内部但方法外部**；属于整个类共享
**实例变量**：定义在 `__init__()` 或其他实例方法中，使用 `self.xxx`；属于某个对象独有


























































































