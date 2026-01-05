## 基础
### Python 基本数据类型
**不可变数据类型**：`int`、`float`、`bool`、`str`、`tuple`
**可变数据类型**：`list`、`set`、`dict`
> Python 中说某个类型“可变”与否，是说**对象的值（内容）能否原地改变**。  
   可变类型如 `list`、`dict` 可以**原地改**；不可变类型如 `int`、`str`、`tuple` **一旦创建就不能改，只能重新创建新对象**。
   给 `int` 类型变量重新赋值，不是修改原对象，而是**创建了一个新对象并重新绑定变量名**

### Python的`is`和 `==` 有什么区别

`is`：**是否是同一个对象**
- 比较的是两个对象的**内存地址是否相同**（是否指向同一个对象）。
- 只有当两个变量引用的是同一个对象时，`is` 才返回 `True`。

`==`：**值（内容）是否相等**
- 比较的是两个对象的**值是否相等**。
- 是通过调用对象的 `__eq__()` 方法实现的。
- 即使是两个不同的对象，只要它们的内容一样，`==` 就返回 `True`。

> **`is` 只用于判断身份，如和 `None` 比较时才推荐用 `is`**
### python字典的key，有什么特征？
- **唯一性**：相同的键只能出现一次
- **不可变性**：键必须是不可变的可hash类型，如字符串，数字或元组。所以，列表不能作为字典的key
- **可比较性**：通过 `==` 比较判断键是否相同
- **高效性**：哈希表结构提供 O(1) 的查找效率

### Python如何判断一个字典里面是否有某个key
#### ①、使用 `in` 关键字
- ✔️ 返回 `True` 说明 key 存在
- ❌ 只能判断 **key**，不能判断 value
#### ②、使用`dict.get(key)` 判断是否为`None`
- 不会抛出异常（key 不存在时返回 `None`）
- 如果 value 本身就是 `None`，这个方法就不准确
### python 的GIL什么作用
#### 介绍
GIL 是 python 的全局解释器锁，**保证同一时间仅有一个线程执行 python 字节码**，**保证线程安全**
这是因为 CPython 的许多底层对象（比如 `list`, `dict`, `int` 等）并不是线程安全的。如果多个线程同时修改对象而没有加锁，容易造成内存错误或数据损坏。
GIL 让解释器**以串行方式执行 Python 代码块**，避免了多线程同时访问共享数据造成的不可预期的问题。
#### GIL 影响
**多线程无法真正并行执行 Python 代码**
虽然 Python 支持多线程（`threading` 模块），但因为 GIL 的存在，Python 多线程在 CPU 密集型任务中并不能实现真正的并行，而是串行切换执行。
- CPU 密集型任务：如计算、加密、图像处理等，**会被 GIL 限制性能**。
- IO 密集型任务：如文件读写、网络请求等，**受 GIL 影响较小**，可以通过线程切换提高效率。
#### GIL 与互斥锁的区别
互斥锁是等一个线程任务结束后，才执行下一个线程，在这期间其他线程是阻塞；
GIL是若线程进入I/O等待，会释放锁给其他线程执行，在这期间线程任务是没有结束的。

### 绕过 GIL 的解决方案
#### 使用多进程
`multiprocessing` 模块可以创建多个进程，每个进程都有自己的 Python 解释器和内存空间，**互不干扰，各自有 GIL，因此可以实现并行计算**。
### 强变量类型什么作用
> 每个变量都有明确的类型，且类型之间不会自动进行隐式转换，除非程序员显式指定。

1. 提高程序安全性
2. 减少隐式类型转换带来的bug
3. 提升 IDE/编译器的提示能力


### python 实现多线程、多进程和协程
#### 1. 多进程：`multiprocessing`模块
适合：**CPU 密集型任务**（如大量计算）
**示例**：
```python
import multiprocessing
import time

def task(name):
    print(f"进程 {name} 开始")
    time.sleep(2)
    print(f"进程 {name} 结束")

if __name__ == "__main__":
    p1 = multiprocessing.Process(target=task, args=("A",))
    p2 = multiprocessing.Process(target=task, args=("B",))
    p1.start()
    p2.start()
    p1.join()
    p2.join()
```
**优点**：
- 每个进程拥有独立 GIL，可以实现真正的并行。
- 更适合 **高 CPU 使用率的程序**。
#### 2. 多线程：`threading`模块
适合：**IO 密集型任务**（如网络请求、文件读写）
**示例**：
```python
import threading
import time

def task(name):
    print(f"线程 {name} 开始")
    time.sleep(2)
    print(f"线程 {name} 结束")

t1 = threading.Thread(target=task, args=("A",))
t2 = threading.Thread(target=task, args=("B",))
t1.start()
t2.start()
t1.join()
t2.join()
```
**注意**：
- **受 GIL 限制**：Python 线程不能并行执行 CPU 密集型任务。
- 多线程更适合 **等待型的任务**（比如网络/磁盘 IO）。
#### 3. 协程：`asyncio`模块
适合：**高并发的 IO 密集型场景**（如爬虫、网络服务）
**示例**：
```python
import asyncio

async def task(name):
    print(f"协程 {name} 开始")
    await asyncio.sleep(2)
    print(f"协程 {name} 结束")

async def main():
    await asyncio.gather(
        task("A"),
        task("B")
    )

asyncio.run(main())
```
**特点**：
- 使用 `async` 定义协程，`await` 挂起等待。
- 单线程下实现**高并发**。
- 资源开销小、切换快，但必须是**异步 IO 场景**。
### python 迭代器、生成器、装饰器？metaclass、args、kwargs
#### 迭代器
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
#### 生成器
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
#### 装饰器
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

#### 元类`metaclass`
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
#### `*args`
`*args`：**表示可变数量的位置参数**（Positional Arguments）
**用途**：可以传入任意多个参数，`args` 会以**元组形式**接收它们。
```python
def func(*args):
    print(args)

func(1, 2, 3)  # 输出: (1, 2, 3)  --> args 是一个元组
```
#### `**kwargs`
`**kwargs`：**表示可变数量的关键字参数**（Keyword Arguments）
**用途**：可以传入任意多个 `key=value` 形式的参数，`kwargs` 会以**字典形式**接收它们。
```python
def func(**kwargs):
    print(kwargs)

func(name='Tom', age=18)  
# 输出: {'name': 'Tom', 'age': 18}  --> kwargs 是一个字典
```
### finally 关键字了解吗，使用时有什么需要注意的？
#### 介绍
`finally` 是 Python 中 `try...except...finally` 异常处理结构的一部分。
它的**作用是无论是否发生异常，都会执行 `finally` 语句块中的代码**，常用于**清理资源**、**关闭文件**、**释放锁**等操作。
#### 注意事项
| ⚠️ 注意点                    | 说明                                                                                               |
| ------------------------- | ------------------------------------------------------------------------------------------------ |
| `finally` 一定会执行           | 无论 try 块中是否发生异常，或有没有 `except` 块，`finally` 都会执行。即使在 try 或 except 中用了 `return`、`break`、`continue`。 |
| 即使抛出未捕获的异常，`finally` 也会执行 | 即使异常没有被 `except` 捕获，`finally` 块仍会运行。                                                             |
| `finally` 中的异常会覆盖原异常      | 如果 `finally` 中又抛出异常，会覆盖原来的异常，造成原始错误信息丢失。                                                         |
| 通常用来做清理工作                 | 比如关闭文件、释放数据库连接、释放锁等。                                                                             |
- 不要在 `finally` 中抛出新的异常，容易掩盖原始异常。
- 避免在 `finally` 中使用 `return`，同样会掩盖之前的返回。

### `with open()` 介绍
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

### Python 类方法和实例方法区别
#### 实例方法（Instance Method）
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
#### 类方法（Class Method）
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

#### 静态方法（@staticmethod）
```python
class Math:
    @staticmethod
    def add(a, b):
        return a + b
```
- 无 `self`、无 `cls`
- 类和实例都能调用
- 与类无关，仅组织逻辑的作用

#### 类变量和实例变量区别
**类变量**：定义在类的**内部但方法外部**；属于整个类共享
**实例变量**：定义在 `__init__()` 或其他实例方法中，使用 `self.xxx`；属于某个对象独有

###  python 面向对象的常用方法，如，`__call__`方法，`__str__`
##### 1. 构造和析构方法
`__init__(self, ...)`：**构造函数（初始化方法）**
用于创建对象时初始化属性
```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
```
`__del__(self)`：析构函数
对象被销毁时调用（不建议重度使用）
```python
def __del__(self):
    print(f"{self.name} 被销毁")
```
##### 2. 魔术方法（Magic Methods）
> 魔术方法是那些名字前后有**两个下划线**（例如 `__init__`、`__str__`、`__add__`）的特殊方法。它们不是我们通常直接调用的，而是由 Python 解释器在**特定情况下自动调用**的

`__str__(self)`：**对象转字符串**
```python
def __str__(self):
    return f"{self.name}, {self.age}岁"
```

`__eq__(self, other)`、`__lt__`、`__gt__`：**重载比较运算符**
```python
def __eq__(self, other):
    return self.age == other.age
```
##### 3. 类方法与静态方法
`@classmethod`
第一个参数是 `cls`，用于操作类本身。
```python
class Person:
    count = 0
    
    @classmethod
    def get_count(cls):
        return cls.count
```

`@staticmethod`
没有默认的 `self` 或 `cls` 参数。
```python
@staticmethod
def say_hello():
    print("Hello!")
```

##### 4. 实例方法
普通方法，第一个参数是 `self`，用于操作实例属性。

##### 5. 属性方法（`@property`）
将方法当作属性来访问，通常用于封装。

##### 6. 特殊用途方法
| 方法名           | 作用                |
| ------------- | ----------------- |
| `__len__`     | 定义 `len(obj)` 的行为 |
| `__getitem__` | 支持索引访问 obj[i]     |
| `__setitem__` | 支持 obj[i] = val   |
| `__iter__`    | 定义可迭代对象           |
| `__next__`    | 定义迭代器的下一个值        |
| `__call__`    | 使对象像函数一样被调用       |

###  `__new__`和`__init__`区别
| 特性       | `__new__`                       | `__init__`               |
| -------- | ------------------------------- | ------------------------ |
| **作用**   | 负责创建对象（分配内存、返回实例）               | 负责初始化对象（设置属性等）           |
| **返回值**  | 必须返回一个实例（通常是 `super().__new__`） | 没有返回值（返回值会被忽略）           |
| **调用时机** | 在对象创建之前（类实例化时最先被调用）             | 在对象创建之后（`__new__` 成功返回后） |
| **常用场景** | 实现**单例模式**、元类控制等高级用途            | 给对象设置初始值是常规使用            |


## 进阶
### 了解协程吗？简述一下Python协程的使用
##### 什么是协程？
> 协程是一种比线程更轻量的并发方式，可以在函数中中断和恢复执行，**适合处理大量 I/O 密集型任务**。

Python 协程基于 `async/await` 语法构建于 **事件循环（event loop）机制**之上。
##### 基本语法
```python
import asyncio

async def say_hello():
    print("Hello")
    await asyncio.sleep(1)
    print("World")

# 启动协程（推荐方式）
asyncio.run(say_hello())
```
##### 常用方法
| 用法                          | 说明         |
| --------------------------- | ---------- |
| `async def`                 | 定义一个协程函数   |
| `await 协程/异步操作`             | 暂停执行，等待结果  |
| `asyncio.run()`             | 启动协程       |
| `asyncio.gather(a, b, ...)` | 并发运行多个协程任务 |
| `asyncio.create_task()`     | 创建后台协程任务   |
##### 并发执行示例
```python
import asyncio

async def task(name, delay):
    print(f"{name} 开始")
    await asyncio.sleep(delay)
    print(f"{name} 结束")

async def main():
    await asyncio.gather(
        task("任务1", 2),
        task("任务2", 1)
    )

asyncio.run(main())
```
输出顺序为：
```bash
任务1 开始
任务2 开始
任务2 结束
任务1 结束
```
说明多个任务**异步并发执行**了，而不是串行等待

##### 协程和线程对比
| 特性    | 协程             | 线程                  |
| ----- | -------------- | ------------------- |
| 开销    | 更小（纯用户态，无系统调度） | 较大（系统级资源调度）         |
| 并发    | I/O 密集优选       | 适合 I/O 和部分 CPU 密集场景 |
| 切换效率  | 快（无需上下文切换）     | 慢（涉及系统调用）           |
| 使用复杂度 | 高（需要事件驱动模型）    | 低                   |
### pyc 文件
#### 什么是 `.pyc`文件
`.pyc` 文件是 Python **源代码 `.py` 文件编译后的字节码文件**
- `.py`：你写的源代码
- `.pyc`：Python 编译器将 `.py` 文件**编译后**生成的 **字节码文件**
- 这些文件通常会存放在项目目录下的 `__pycache__` 文件夹中。
#### 生成`.pyc`文件的过程
**导入一个模块**（不是直接运行），Python 会执行以下步骤：
1. 读取 `.py` 源代码；
2. 编译成字节码（`.pyc`）；
3. 将 `.pyc` 缓存到 `__pycache__` 目录中；
4. 下次再导入该模块时，会**优先加载 `.pyc`**，加快执行速度（如果源代码没改）。
>**简单来说，Python 的运行过程可以概括为：** 
>源代码 `.py` -> (编译) -> 字节码 `.pyc` -> (执行) -> Python 虚拟机（PVM）
#### `.pyc`的好处
| 优点     | 说明                             |
| ------ | ------------------------------ |
| 加快加载速度 | 下次导入模块时可以直接加载 `.pyc`，省去编译过程    |
| 编译中间产物 | 是 Python 源码到机器执行之间的中间步骤        |
| 可跨平台   | 只要 Python 版本一致，`.pyc` 可在不同系统运行 |
#### 注意事项
1. `.pyc` 文件 **不是二进制机器码**，只是 Python 虚拟机可以识别的字节码。
2. 如果你修改了 `.py` 源码，Python 会自动判断是否重新编译生成新的 `.pyc`。
3. 删除 `.pyc` 没关系，Python 会在需要时重新生成。
4. **不要手动编辑 `.pyc`**，内容是二进制，非人类可读。
### python gc的实现
#### 什么是垃圾回收？
> **垃圾回收机制就是自动释放程序中不再使用的内存空间**，防止内存泄漏。

#### GC 的两种机制
| 名称                           | 简介                         | 特点              |
| ---------------------------- | -------------------------- | --------------- |
| **引用计数（Reference Counting）** | 每个对象都维护一个引用计数器，引用为0时，会回收对象 | 快速、实时，但无法处理循环引用 |
| **分代垃圾回收（Generational GC）**  | 专门用于处理引用计数无法回收的对象          | 解决循环引用问题，按代优化性能 |

##### 1. 引用计数机制
每个对象都有一个引用计数器：
- 每增加一个引用，计数 +1
- 每删除一个引用，计数 -1
- 当计数为 0 时，自动释放内存
**缺点：无法处理循环引用**
```python
l = []
l.append(l)  # 自己引用自己，形成循环引用
del l        # 这里只减少一次引用计数，但还存在一条引用链
```
##### 2. 分代回收机制（Generational GC）
为了解决循环引用，Python 引入了**分代回收机制**。
**原理**：
- 把对象按“存活时间”分为 3 代：**0 代、1 代、2 代**
- 新创建的对象属于第 0 代
- 如果对象存活得久，会晋升到更高代
- **GC 会更频繁地回收年轻代对象**

### 引用计数降低的时机 （不会，面试官给了答案是对象删除时，确认下）
Python 中每次对象的引用被“**移除**”或“**失效**”，它的引用计数就会 **-1**。
**注意：引用计数 ≠ 真正释放**
即使引用计数降为 0：
- 如果对象有 `__del__()` 方法，可能延迟清理
- 如果在 GC 检查周期内未触发，也可能暂时保留


### 什么是单例模式？用Python怎么实现
#### 单例模式介绍
**单例模式**是一种常用的设计模式，其核心思想是：
**整个程序生命周期中，某个类只创建一个实例，并对外提供统一的访问方式。**
#### Python实现单例常用方法
##### 方式1：使用类变量缓存实例（最常用）
```python
class Singleton:
    _instance = None  # 类变量，保存唯一实例

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
	        # 调用默认继承`object`类中的`__new__()`方法
            cls._instance = super().__new__(cls)
        return cls._instance

# 测试
a = Singleton()
b = Singleton()
print(a is b)  # ✅ True，表示是同一个对象
```
> 使用 `__new__()` 是 Python 中最推荐的标准写法，线程安全的话需加锁处理。

##### 方式2：使用装饰器实现单例
```python
def singleton(cls):
    instances = {}

    def wrapper(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]

    return wrapper

@singleton
class MyClass:
    pass

a = MyClass()
b = MyClass()
print(a is b)  # ✅ True
```
> 适用于多个类都想变成单例的情况，复用性好。

##### 方式3：使用模块实现天然单例
Python 的模块本身就是单例的！
```python
# config.py
db_config = {'host': 'localhost', 'port': 3306}

# main.py
import config
print(config.db_config)
```
> 模块在第一次导入后，会被缓存在 `sys.modules` 中，之后无论导入多少次，都是**同一个模块实例**。

##### 方式4：使用元类（高级写法）
```python
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class MyClass(metaclass=SingletonMeta):
    pass

a = MyClass()
b = MyClass()
print(a is b)  # ✅ True
```
> 面向对象高手常用写法，可用于多个类共享单例逻辑。

###  python 深浅拷贝
**浅拷贝（Shallow Copy）**：复制最外层对象，**内部嵌套对象仍然引用原来的**
**深拷贝（Deep Copy）**：复制对象的所有层级，**原对象与副本完全独立**




  

