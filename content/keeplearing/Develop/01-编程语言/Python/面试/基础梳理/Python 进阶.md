## Python 的GIL什么作用
### 介绍
GIL 是 python 的全局解释器锁，**保证同一时间仅有一个线程执行 python 字节码**，**保证线程安全**
这是因为 CPython 的许多底层对象（比如 `list`, `dict`, `int` 等）并不是线程安全的。如果多个线程同时修改对象而没有加锁，容易造成内存错误或数据损坏。
GIL 让解释器**以串行方式执行 Python 代码块**，避免了多线程同时访问共享数据造成的不可预期的问题。
### GIL 影响
**多线程无法真正并行执行 Python 代码**
虽然 Python 支持多线程（`threading` 模块），但因为 GIL 的存在，Python 多线程在 CPU 密集型任务中并不能实现真正的并行，而是串行切换执行。
- CPU 密集型任务：如计算、加密、图像处理等，**会被 GIL 限制性能**。
- IO 密集型任务：如文件读写、网络请求等，**受 GIL 影响较小**，可以通过线程切换提高效率。
### GIL 与互斥锁的区别
互斥锁是等一个线程任务结束后，才执行下一个线程，在这期间其他线程是阻塞；
GIL是若线程进入I/O等待，会释放锁给其他线程执行，在这期间线程任务是没有结束的。
### 绕过 GIL 的解决方法
#### 1. 使用多进程
`multiprocessing` 模块可以创建多个进程，每个进程都有自己的 Python 解释器和内存空间，**互不干扰，各自有 GIL，因此可以实现并行计算**。
#### 2. Python 3.14 free-threaded 

## Python 上下文管理器
**Python 上下文管理器**：用 with 语句管理的资源自动进入和退出。
**核心作用**：保证资源（文件、锁、连接等）在使用完毕后一定被正确释放，即使发生异常。
### 基本概念与协议
上下文管理器本质上是实现了`__enter__`与`__exit__`方法的对象。`with`语句在进入代码时调用`__enter__`，在离开代码块时无论是否发生异常都会调用`__exit__`。
```python
class MyCtx:
    def __enter__(self):
        print("enter")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("exit")
        return False   # False 表示异常不被吞掉
```
使用时：
```python
with MyCtx() as ctx:
    ...
```
### 异常控制语义
`__exit__(exc_type, exc_val, exc_tb)` 的返回值决定异常是否被抑制。返回 `True` 表示异常已被处理，异常不会向外抛出；返回 `False` 或 `None` 表示异常继续传播。这一机制使上下文管理器既可以做清理，也可以承担异常屏蔽的职责。

## Python 协程
### 介绍
> 协程是一种比线程更轻量的并发方式，可以在函数中中断和恢复执行，**适合处理大量 I/O 密集型任务**。

Python 协程基于 `async/await` 语法构建于 **事件循环（event loop）机制**之上。
### 基本语法
```python
import asyncio

async def say_hello():
    print("Hello")
    await asyncio.sleep(1)
    print("World")

# 启动协程（推荐方式）
asyncio.run(say_hello())
```
### 常用方法
| 用法                          | 说明         |
| --------------------------- | ---------- |
| `async def`                 | 定义一个协程函数   |
| `await 协程/异步操作`             | 暂停执行，等待结果  |
| `asyncio.run()`             | 启动协程       |
| `asyncio.gather(a, b, ...)` | 并发运行多个协程任务 |
| `asyncio.create_task()`     | 创建后台协程任务   |
### 示例
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
```text
任务1 开始
任务2 开始
任务2 结束
任务1 结束
```
说明多个任务**异步并发执行**了，而不是串行等待
### 在协程函数中执行同步代码会发生什么
在 Python 的协程函数(`async def`)中执行同步代码，本身不会报错，但在运行语义上会**阻塞事件循环**，使异步程序失去并发能力，这是异步系统中最典型、也是最危险的问题之一。
可以从事件循环的工作机制来理解这一点。

`asyncio` 采用**单线程协作式调度**模型。只有当协程执行到 `await` 时，才会主动让出控制权，事件循环才能切换去执行其他协程。
如果在协程内部执行的是同步阻塞代码，例如：
- `time.sleep()`
- 同步网络请求（如 `requests.get`）
- 同步数据库驱动
- CPU 密集型计算
那么在这段代码执行期间：
- 当前协程不会 `await`
- 事件循环无法切换任务
- 所有其他协程全部被“卡住”
结果就是：**异步程序退化为串行程序**。

## 介绍一下 Python 的 gc 垃圾回收机制
### 什么是垃圾回收
**垃圾回收机制就是自动释放程序中不再使用的内存空间**，防止内存泄漏。
### GC 的两种机制
| 名称                           | 简介                         | 特点                  |
| ---------------------------- | -------------------------- | ------------------- |
| **引用计数（Reference Counting）** | 每个对象都维护一个引用计数器，引用为0时，会回收对象 | 快速、实时，但无法处理循环引用     |
| **分代垃圾回收（Generational GC）**  | 专门用于处理引用计数无法回收的对象          | 解决**循环引用**问题，按代优化性能 |
#### 1. 引用计数
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
#### 2. 分代垃圾回收
为了解决循环引用，Python 引入了**分代回收机制**。
**原理**：
- 把对象按“存活时间”分为 3 代：**0 代、1 代、2 代**
- 新创建的对象属于第 0 代
- 如果对象存活得久，会晋升到更高代
- **GC 会更频繁地回收年轻代对象**
### 引用计数降低的时机
Python 中每次对象的引用被“**移除**”或“**失效**”（也可以说是在对象删除时），它的引用计数就会 **-1**。
**注意：引用计数 ≠ 真正释放**
即使引用计数降为 0：
- 如果对象有 `__del__()` 方法，可能延迟清理
- 如果在 GC 检查周期内未触发，也可能暂时保留
## Python 内存管理模型
Python 的内存管理模型是“解释器级对象管理 + 操作系统级内存分配”的双层结构，其核心目标是**在保证开发便利性的同时，尽量降低频繁分配与释放带来的性能开销**。
### 1. 对象级管理：引用计数为核心
CPython 采用引用计数作为主要的垃圾回收机制。
**过程**：
每个对象维护一个引用计数器，当有新的引用指向该对象时计数加一，引用消失时减一；当计数归零时，对象立即被销毁并释放其占用的内存。
**优势**：
对象生命周期确定、资源释放及时，但缺陷是无法处理循环引用。
### 2. 循环引用的补充机制：分代垃圾回收
为弥补引用计数无法处理环的不足，CPython 引入了分代 GC，将对象分为三代。新创建对象位于第 0 代，存活一次回收后进入第 1 代，再存活进入第 2 代。回收频率从 0 代到 2 代逐级降低。该 GC 只关注容器对象（list、dict、set、自定义对象等），并通过遍历对象引用关系，检测并清理仅被循环引用持有的对象。
### 3. 内存分配层：pymalloc 与内存池
对象被销毁并不等价于立即归还操作系统。CPython 在内部实现了 `pymalloc` 小对象分配器，将小于 512 字节的内存请求交由其管理。pymalloc 采用“arena → pool → block”三级结构，将内存切分为固定尺寸块缓存起来，后续同尺寸对象可直接复用，避免频繁向系统 `malloc` / `free`。
### 4. 内存释放的现实行为
即使对象被销毁，其占用的内存往往只归还给 Python 解释器的内存池，而非操作系统。因而进程 RSS 常表现为“只增不减”，这在长时间运行的服务中非常典型。这并非内存泄漏，而是内存复用策略的自然结果。
### 5. 工程层面的影响
大量创建与销毁小对象通常不会引发系统层面的内存抖动，但若出现大量长生命周期对象或不可达循环引用，则会表现为内存持续增长。此时需要借助 `gc` 模块、`tracemalloc`、对象快照等工具定位引用关系与泄漏源。

**Python 的内存管理并非单一机制，而是引用计数、分代回收与内存池复用共同作用的结果。**

## 什么是单例模式？Python 中如何实现
### 介绍
**单例模式**是一种常用的设计模式，其核心思想是：
**整个程序生命周期中，某个类只创建一个实例，并对外提供统一的访问方式。**
### 单例实现常用方法
#### 方式1：使用类变量缓存实例（最常用）
**使用 `__new__()` 是 Python 中最推荐的标准写法，线程安全的话需加锁处理。**
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

#### 方式2：使用装饰器实现单例
**适用于多个类都想变成单例的情况，复用性好。**
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

#### 方式3：使用模块实现天然单例
Python 的模块本身就是单例的！
模块在第一次导入后，会被缓存在 `sys.modules` 中，之后无论导入多少次，都是**同一个模块实例**。
```python
# config.py
db_config = {'host': 'localhost', 'port': 3306}

# main.py
import config
print(config.db_config)
```
#### 方式4：使用元类（高级写法）
面向对象高手常用写法，可用于多个类共享单例逻辑。
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



























