有缓冲channel和无缓冲channel的区别，底层原理  
select怎么用  
gmp模型

make 和 new有什么区别  
defer的执行时机  
defer 常见的用法  
panic 怎么处理  
协程发生阻塞的情况有哪些？  
channel满了 消费者和生产者会怎么样。 对值为nil的channel读取会发生什么？  
map的底层结构是什么样的  
map是并发安全的吗？ sync.Map  
map的遍历是有序的还是无需的？ 如果需要实现有序的遍历如何做（不知道这个有啥意义）  
GMP是什么？介绍一下。M和P的数量是怎么指定  


sync 包中的 map，once，pool，waitgroup怎么实现的  
sync.map中dirty map咋升成read map的
创建新切片，设置多少大小合适
切片扩容原理
gc三色标记讲讲
gmp 模型
select
哪些方式可以控制goroutine数量
gc，又追问写屏障怎么工作的

切片和数组的区别
goroutine和线程，进程区别
go内存泄漏排查思路

## 基础
### 为什么选择golang？go语言相比其他语言的特点和优势
1. **原生支持高并发**（goroutine + channel）：Go 的 **并发编程模型非常强大**，基于 CSP（通信顺序进程）模型，使用 `goroutine` 可以轻松开启数万个轻量级线程。
2. **执行性能强**，接近 C/C++：编译型语言
3. **部署简单**，仅需一个可执行文件：编译后直接生成一个可执行文件，没有依赖，跨平台方便
4. **语法简单**，容易维护
5. **标准库强大、工具链完善**：go mod 包管理工具好用，还支持交叉编译
6. **开源生态好**，具有众多第三方库：gin, echo, gorm, xorm, viper,

### golang里面的指针和c++里面的指针有什么区别？
1. Go 中**没有指针运算，更安全**【**只允许访问、修改**】：不能对指针做 `p++`，`p--`等操作，不能做内存地址偏移。
2. **无需手动释放内存，有 GC 管理**
3. 使用场景主要是**改值**和**传引用**
### defer 
#### 是什么
`defer`表示：**推迟执行一个函数，直到外围函数返回时才执行，且执行顺序是“先进后出（LIFO）”**。
#### 原理
`defer`底层原理：
1. 编译器在函数编译期将 `defer` 转换为 **运行期插入调用表**；
2. 函数运行时，如果遇到 `defer`，会将相关函数参数 **先计算好并压栈**；
3. 外围函数返回前，按 **“先进后出”** 的顺序执行 defer 函数。
重点：
> **defer 的参数在 defer 注册时就计算完成，而不是在执行时再算**！
####  defer 里是否可以改变参数
1.**不可以改变已传入的参数 —— 因为参数值是“值拷贝”**
```go
// 示例：修改参数无效
func test() {
    x := 10
    defer fmt.Println("defer:", x)
    x = 20
    fmt.Println("now:", x)
}
```
结果为：
```bash
now: 20
defer: 10
```
解释：`defer fmt.Println("defer:", x)` 注册时 `x=10`，那一刻参数就固定了。

2.**可以通过引用类型来间接修改**
```go
// 示例：用指针或引用类型（如 slice/map）可以生效
func test() {
    x := 10
    defer func(p *int) {
        fmt.Println("defer:", *p)
    }(&x)

    x = 20
    fmt.Println("now:", x)
}
```
结果为：
```bash
now: 20
defer: 20
```
解释：传的是 `&x`，不是 `x` 值本身，所以 `defer` 执行时看到的是修改后的值。

#### defer 语句在 return 前还是后
> **`defer` 是在 `return` 设置返回值后、函数退出前执行**，  若你想让 `defer` 改变返回结果，就需要用“命名返回值”。

#### 执行顺序
**先进后出**（LIFO）

#### 注意事项
1. **参数在 defer 定义时就求值了（不是执行时）**
```go
func test() {
	x := 10
	defer fmt.Println("defer x:", x)
	x = 20
	fmt.Println("current x:", x)
}
// 输出：current x: 20   defer x: 10
// 解释：defer 定义时 `x` 是 10，所以打印的是 10。
```
2. **可修改命名返回值（技巧！）**
```go
func demo() (result int) {
	defer func() {
		result += 10
	}()
	return 5
}
// 输出：15
// 解释：命名返回值 `result` 是一个变量，`defer` 修改的是这个变量。
```

3. **defer 会捕获闭包外的变量（引用效果）**
```go
func test() {
	x := 1
	defer func() {
		fmt.Println("x =", x)
	}()
	x = 2
}
// 输出：x = 2
// defer 中的闭包引用的是变量 `x`，所以输出的是更新后的值。
```

4. **defer 是函数级的：只在当前函数返回时触发**
```go
func inner() {
	defer fmt.Println("inner done")
}
func outer() {
	defer fmt.Println("outer done")
	inner()
}
// 输出：inner done 
//      outer done
// 每个函数的 defer 只在自己函数返回时触发。
```

5. **defer 有运行开销（特别是大量使用）**
- 每次 defer 都会有一次额外调用栈开销；
- 在循环中频繁使用 defer（如 `for` 中 defer close）要谨慎；
- 可用手动 close 替代（更高性能）。

#### 常见用法
- **关闭文件**：`defer file.Close()`
- **解锁**：`defer mu.Unlock`
- **捕获 panic** ：`defer recover`
- **网络连接关闭**：`defer conn.Close()`

### 内存溢出和内存泄露分别是什么?
##### 内存溢出（Out of Memory）
程序申请内存时，**超出了系统可用内存的限制**，导致系统或进程崩溃。
##### 内存泄露（Memory Leak）
程序中**不再使用的内存没有被释放**，但仍然被引用着，导致内存占用越来越高。

### go 异常处理
#### 错误处理机制：
**1. 普通错误处理：使用`error`接口**
Go 中推荐使用显式返回 `error`值的方式来处理常规错误，这也是为什么很多人诟病的写 go 时要经常写`if err != nil{}`
**2. panic：类似异常抛出**
当遇到**严重错误**（如数组越界、空指针）时，Go 会触发 panic。
- `panic` 会中断程序执行
- 沿着调用栈逐层向上传播，直到程序终止

**3. recover：拦截 panic，防止程序崩溃**
`recover` 必须搭配 `defer` 使用，用于 **从 panic 中恢复**。
```go
func safeRun() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("捕获 panic：", r)
		}
	}()
	panic("程序崩溃了！")
}
// 输出：捕获 panic： 程序崩溃了！
```

### slice
#### Go 中 new 和 make 的区别
> `new` 用于**分配内存，返回指针，值为零**；`make` 用于**初始化内置类型 slice、map、channel，返回已初始化的可用值**。
- `new` = 分配 + 零值指针
- `make` = 初始化 + 可直接用

#### Go 的数组和切片的区别。什么时候用数组什么时候用切片?
##### 数组（Array）
- **固定长度**，长度是类型的一部分，比如 `[5]int` 和 `[10]int` 是不同类型。
- **值类型**，赋值或传参会拷贝整个数组。
- 长度不可变，定义时长度必须确定。
- 内存连续分配。
```go
var arr [3]int          // 声明一个长度为3的int数组，默认值为0
arr[0] = 1
fmt.Println(arr)        // 输出: [1 0 0]
```
**适用场景**：
1. 元素个数固定且不会变化
2. 作为函数参数传递且不想修改原始数据

##### 切片（Slice）
- 是对数组的**动态、灵活的抽象**，本质是一个结构体，包含：指向数组的指针、长度和容量。
- **长度可变**，可以扩容（`append`）。
- 是引用类型，赋值和传参传的是切片结构体，底层数组指针是共享的。
- 可以从数组或其他切片通过切片操作生成。
```go
s := []int{1, 2, 3}     // 声明并初始化切片
s = append(s, 4)        // 动态添加元素
fmt.Println(s)          // 输出: [1 2 3 4]
```
**适用场景**：
1. 需要动态增删元素或元素数量不固定
2. 大多数日常开发

#### 切片的扩容原理
##### 切片的底层机构
Go 切片的本质是一个结构体，定义如下：
```go
type slice struct {
    ptr *T    // 指向底层数组的指针
    len int   // 当前切片的长度
    cap int   // 当前切片的容量
}
```
所以：**切片本身是一个小对象，拷贝不会拷贝底层数组**
##### 扩容原理
切片扩容是通过 `append()` 函数触发的，**当容量不够时，Go 会自动分配一个新的、更大的数组，并将原来的内容复制过去**。
扩容逻辑大致如下（Go 源码实现简化）：
```go
如果 新长度 <= 2 * 原容量：
    新容量 = 2 * 原容量
否则：
    新容量 = 新长度
```
- 小切片容量 < 1024 会**按2倍扩容**
- 大切片（容量 >1024）可能按**1.25倍**扩容（Go 1.17+有更细优化）
- 扩容后**重新分配底层数组，旧数组内容复制过去**

##### Go中切片扩容后地址变了吗
会变。因为容量不够时，Go 会创建一个新的底层数组，**拷贝旧数据**过去，所以底层地址肯定变了！

### Channel
#### channel 介绍、用法、底层原理
##### Channel 是什么
 >`channel`是 Go 的一种**引用类型**，用于在`Goroutine`之间**传递数据**，可以理解为线程安全的管道。
> `channel` 是 `goroutine` 之间通信的核心机制，内置同步、阻塞、线程安全，支持缓冲和选择，背后由 `Go runtime` 实现调度和挂起机制。

- 用于**发送（send）**   和 **接收（receive）** 数据
- 保证发送和接收之间的同步（同步通信模型）
- channel 自带阻塞行为，是一种天然的“锁”
##### 基本用法
**1.声明和初始化**
```go
ch := make(chan int) // 创建一个 int 类型的无缓冲 channel
```
**2.发送和接收**
```go
ch <- 42      // 发送数据（写）
val := <-ch   // 接收数据（读）
```
默认是**阻塞的**：
- 发送：等待有人接收才继续
- 接收：等待有值可以读

**3.带缓冲 channel（不阻塞）**
```go
ch := make(chan int, 3) // 创建带缓冲的 channel，容量为 3

ch <- 1 // 不阻塞，直到写满缓冲区
```
特点：
- 当缓冲区满：**发送方阻塞**
- 当缓冲区空：**接收方阻塞**

**4.关闭 channel**
```go
close(ch)      // 关闭 channel（只能由发送方关闭）

val, ok := <-ch // ok=false 表示 channel 已关闭
```
⚠️ 关闭后再写入 → panic  
⚠️ 读可以继续读，直到 channel 被耗尽

**5.配合 select 使用（监听多个 channel）**
```go
select {
case msg := <-ch1:
    fmt.Println("收到：", msg)
case ch2 <- 99:
    fmt.Println("发送成功")
default:
    fmt.Println("无事可做")
}
```


##### 底层原理
`channel`是 Go runtime 层的结构，它的核心数据结构如下（简化）：
```go
type hchan struct {
    qcount   uint      // 队列中元素数量
    dataqsiz uint      // 缓冲区大小
    buf      unsafe.Pointer // 缓冲区指针
    sendx    uint      // 发送索引
    recvx    uint      // 接收索引
    recvq    waitq     // 接收等待队列
    sendq    waitq     // 发送等待队列
    lock     mutex     // 互斥锁
}
```
**关键机制：**
- **无缓冲 channel**：发送/接收必须一对一匹配，否则阻塞。使用 `recvq` 和 `sendq` 来挂起等待的 goroutine。
- **带缓冲 channel**：内部维护一个循环队列（FIFO），由 `buf`, `sendx`, `recvx` 维护写读位置。
- 所有操作都加了锁，channel 是并发安全的。
- `runtime` 会在发送/接收不满足时让当前 goroutine `park`（阻塞），当条件满足后 `unpark`（唤醒）。

#### 什么时候chanal会发生panic
> channel 的 panic 通常发生在**对已关闭或未初始化（nil）的 channel 进行写或关闭操作**，牢记“**只能关闭一次，只能由发送方关闭，不能向已关闭通道发送数据**”。

**1.向已关闭的 channel 发送数据**
```go
ch := make(chan int)
close(ch)
ch <- 1 // ❌ panic: send on closed channel
```
- **向关闭的 channel 写数据会 panic**
- **读取关闭的 channel 是合法的**（会返回零值）

**2.重复关闭 channel**
```go
ch := make(chan int)
close(ch)
close(ch) // ❌ panic: close of closed channel
```
- 一个 channel **只能被关闭一次**
- **一般只有发送方负责关闭 channel**

**3.关闭一个 `nil` 的 channel**
```go
var ch chan int // 默认是 nil
close(ch) // ❌ panic: close of nil channel
```
- 不能对 `nil` channel 做任何操作（包括关闭、读、写）
- 所有操作都会 **永久阻塞** 或 panic（取决于操作类型）

**4.在未初始化的 channel 上发送或接收 -> 可能导致永久阻塞（不是 panic）**
```go
var ch chan int // nil channel
ch <- 1         // ❌ 永久阻塞（不会 panic，但程序卡死）
```
- **nil channel** 的读写操作会**阻塞当前 goroutine**
- 这种问题不容易排查，容易导致“程序假死”

##### 使用 channel 时的安全建议
| 建议                                             | 理由                     |
| ---------------------------------------------- | ---------------------- |
| 只让发送方关闭 channel                                | 避免接收方意外关闭              |
| 在关闭 channel 前检查是否为 nil                         | 避免关闭 nil channel       |
| 使用 `ok` 判断 channel 是否关闭                        | `val, ok := <-ch` 安全接收 |
| 如果是多 goroutine 写入 channel，使用 sync.Once 或控制退出流程 | 避免竞态关闭 channel         |
#### panic 怎么处理
 >Go 中用 `panic` 表示**程序运行时遇到的严重问题**，用 `recover` **捕获 panic 防止崩溃**，但推荐尽量通过返回 `error` 来处理可控错误。

使用 `recover`
```go
func safeRun() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("捕获 panic:", r)
		}
	}()

	// 可能 panic 的代码
	panic("出错了！")
}
```
- 只能在 **`defer` 函数中使用**
- 能捕获到 `panic` 的值，并**阻止程序崩溃**

`panic` + `recover`的调用流程图
```go
函数开始执行
    ↓
可能 panic
    ↓
是否有 defer-recover？
    ↓ 是
捕获并处理错误，程序继续运行
    ↓ 否
程序崩溃，打印调用栈，退出
```
**使用建议：**
- **能返回 error 就不要 panic**（panic 是终止级别）
- `recover()` 只对当前 `goroutine` 的 `panic` 有效
- 一般在库或框架中使用 `recover` 包裹入口，防止程序崩掉




#### channel满了 消费者和生产者会怎么样。 对值为nil的channel读取会发生什么？
**带缓冲 channel**
**channel 满：**
**生产者阻塞**，等待消费者消费一个元素后才可以继续写入
不会 panic（不是异常），但会卡死当前 goroutine（陷入等待）
**channel空：**
**消费者阻塞**，阻塞直到有数据可读

```go
var ch chan int // 默认为 nil
val := <-ch     // ❗ 会“永久阻塞”
```
- **读取 nil channel 会永久阻塞**，不会 panic，但该 goroutine 卡住不动
- 同样，**向 nil channel 写入也会永久阻塞**
- **关闭 nil channel 会 panic**


#### chan中无缓冲和有缓冲的区别
> **无缓冲 channel 是同步通信**，发送必须等接收；**有缓冲 channel 是异步通信**，可先写入缓冲队列。
##### 无缓冲 channel
```go
ch := make(chan int) // 无缓冲
```
特点：
- **同步通信**：发送操作会**阻塞**，直到另一个 goroutine **接收该值**
- 发送和接收必须“手拉手”才能继续
##### 有缓冲 channel
```go
ch := make(chan int, 3) // 容量为3的缓冲通道
```
特点：
- **异步通信**：发送操作在**缓冲区未满**时不会阻塞。
- 一旦缓冲区满了，再发送会阻塞。
- 接收时，如果缓冲区非空，可以立即读到数据。
#### chan主要应用场景
| 场景           | 示例                 |
| ------------ | ------------------ |
| goroutine 通信 | worker 向主线程发送数据    |
| 任务调度         | 生产者/消费者模型          |
| 控制协程退出       | 使用 `chan struct{}` |
| 超时控制         | 配合 `time.After` 使用 |
| 广播通知         | 配合多个协程统一退出         |

#### go 的 select
> `select` 是 Go 中用于 **监听多个 channel 操作（发送/接收）** 的语法结构。  
   它可以在多个 `channel` 中 **随机选择一个可用的分支执行**。
```go
select {
case val := <-ch1:
    fmt.Println("收到 ch1 的数据：", val)
case ch2 <- 100:
    fmt.Println("往 ch2 发送数据")
default:
    fmt.Println("都没准备好")
}
```
- **每个 case** 要么是 `<-chan`（读），要么是 `chan<-`（写）
- **只会执行一个就绪的 case**（多个就绪时随机选一个）
- 如果都不满足，又有 `default`，就执行 `default`
- 如果都不满足、也没有 `default`，会 **阻塞等待**
> `select` 是 Go 并发编程中监听多个 channel 的神器，**一次只执行一个 case，多个就绪时随机选一个，否则阻塞**。


#### 如何判断 channel 是否已满/空（虽然不能直接判断）
> Go 不推荐你显式判断 channel 是否满或空，而是通过 `select + default` 来实现**非阻塞、安全地读写判断**。`len/ch` 仅适合**非并发、调试、辅助逻辑**。

##### 非阻塞写入：判断是否能写
```go
select {
case ch <- val:
    fmt.Println("写入成功")
default:
    fmt.Println("channel 已满，写入失败")
}
```
##### 非阻塞读取：判断是否有值可读
```go
select {
case val := <-ch:
    fmt.Println("读取成功：", val)
default:
    fmt.Println("channel 是空的")
}
```
#### 如何避免因为 channel 阻塞导致协程泄漏
##### 协程泄露
> 当一个 goroutine 启动后因为某些原因 **一直不能退出**（如永久阻塞在 channel 上），就叫 **协程泄漏**

##### 如何避免
> 要避免 goroutine 泄漏，关键是**控制 channel 的读写阻塞行为**，并且使用 **`context` 或 `select` 做好退出控制**，不要让 goroutine 永远等不到数据或机会。

**1. 使用 `select + default` 实现非阻塞操作**
```go
select {
case ch <- val:
    // 写入成功
default:
    // channel 满，丢弃或处理降级逻辑
}
```
**2. 使用 `context.Context` 控制 goroutine 生命周期**
```go
func worker(ctx context.Context, ch <-chan int) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("worker 退出")
            return
        case data := <-ch:
            fmt.Println("处理数据:", data)
        }
    }
}
```
> `context.WithCancel`, `context.WithTimeout` 是常用手段来取消协程

**3. 发送方判断接收方是否还活着（可选）**
```go
ch := make(chan int)

go func() {
    for v := range ch {
        fmt.Println("消费", v)
    }
    fmt.Println("消费者退出")
}()

ch <- 1
close(ch) // 消费者读完后自动退出
```
注意：**关闭 channel 后 range 会自动结束**

**4. 设置缓冲区，减少写阻塞（但不是解决根本问题）**
```go
ch := make(chan int, 100) // 带缓冲，写入时更不易阻塞
```
虽然可以减少阻塞，但最终仍需要消费者消费，**不能根本避免泄漏**

**5. 读取方增加超时控制（防止永久卡读）**
```go
select {
case val := <-ch:
    fmt.Println("收到：", val)
case <-time.After(1 * time.Second):
    fmt.Println("超时退出")
}
```

### Map
#### 哈希表的原理，如何解决哈希冲突
##### 原理
哈希表（Hash Table）是一种基于 **数组 + 哈希函数** 实现的 **键值对存储结构**，它通过将 **键（Key）通过哈希函数映射为数组下标（索引）**，从而能在 **常数时间（O(1)) 内完成插入、查找和删除**。
核心组成：
1. 哈希函数
	1. 将任意长度的 key 映射为固定范围的整数索引
2. 数组
	1. 存储映射后的数据（key-value 对）
##### 哈希冲突
多个不同的 key 映射到了同一个索引位置，就叫做 **哈希冲突**。
##### 哈希冲突解决方法
**① 拉链法**
- 每个数组位置不只存储一个元素，而是一个 **链表或链表结构**。
- 所有哈希值相同的元素都放到这个链表中。
```go
hash("apple") → index 2 → ["apple"]
hash("banana") → index 2 → ["apple", "banana"]
```
**优点**：实现简单，冲突处理能力强
**缺点**：如果冲突过多，会退化为链表，查找效率变成 O(n)

**② 开放地址法**
- 冲突时，尝试在数组中寻找下一个空位。
- 查找方式有几种策略：
**线性探测**：
```go
index = hash(key) % size
if occupied:
    index = (index + 1) % size
```
**二次探测**：
```go
index = (hash(key) + i^2) % size
```
**双重哈希**：
```go
index = (hash1(key) + i * hash2(key)) % size
```

**优点**：不需要额外空间（不像链表）
**缺点**：删除操作复杂；装载因子（load factor）高时性能下降严重

#### Go 的 Map 是有序还是无序的？
Map 是无序的。每次遍历的顺序都是随机的，即使 map 的内容没有变，遍历顺序也可能不同。
**为什么无序？**
> Go 语言的 `map` 底层是通过 **哈希表（hash table）** 实现的，为了安全和效率，Go 会在运行时对哈希遍历做**随机打乱（randomized iteration）**，以防止依赖顺序引发的错误逻辑和安全问题（如哈希碰撞攻击等）。

#### Go 的 Map 如何实现有序访问？(使用切片存储键+sort, for循环切片取map)
##### 法一：提取 key 将其保存至切片，对其排序，然后遍历切片取出 map value
##### 法二：创建 k, v 结构体，将 map 中的数据取出来保存至结构体切片，然后排序

#### Go中map并发安全吗？替代方案？怎么实现并发安全？
Go 的原生 `map` 是为单线程设计的，在多个 goroutine 同时对 map 进行 **写操作或读写混合操作** 时，会出现 **竞态条件**，甚至触发运行时 panic
##### 解决方案：
**方式一：使用`sync.Mutex`手动加锁**
适用于读写比例差不多或写操作较多的场景。
```go
import (
	"sync"
)

type SafeMap struct {
	mu sync.Mutex
	m  map[string]int
}

func (s *SafeMap) Set(key string, value int) {
	s.mu.Lock()
	defer s.mu.Unlock()
	s.m[key] = value
}

func (s *SafeMap) Get(key string) (int, bool) {
	s.mu.Lock()
	defer s.mu.Unlock()
	val, ok := s.m[key]
	return val, ok
}
```

**方法二：使用`sync.RWMutex`支持读写锁分离**
适用于**读操作远多于写操作**的场景。
```go
type SafeMapRW struct {
	mu sync.RWMutex
	m  map[string]int
}

func (s *SafeMapRW) Get(key string) (int, bool) {
	s.mu.RLock()
	defer s.mu.RUnlock()
	val, ok := s.m[key]
	return val, ok
}

func (s *SafeMapRW) Set(key string, value int) {
	s.mu.Lock()
	defer s.mu.Unlock()
	s.m[key] = value
}
```

**方法三：使用 Go 官方推出的并发安全的 `sync.Map`**
Go 官方提供的并发 map 类型，适合**不频繁写入但读多的场景**。
```go
import "sync"

var m sync.Map

m.Store("foo", 42)               // 写入
v, ok := m.Load("foo")           // 读取
m.Delete("foo")                  // 删除
m.Range(func(k, v interface{}) bool {
	fmt.Println(k, v)
	return true
})
```

#### map的底层结构是什么样的
##### 1. `map`的核心结构体：`hmap`
```go
type hmap struct {
	count     int            // 当前元素个数
	flags     uint8          // 标志位（扩容、清理状态等）
	B         uint8          // 2^B = 桶的数量
	noverflow uint16         // 溢出桶数量估算
	hash0     uint32         // 哈希种子（防止碰撞攻击）

	buckets    unsafe.Pointer // 指向桶数组，长度为 2^B
	oldbuckets unsafe.Pointer // 扩容时使用的旧桶数组（增量扩容）
	nevacuate  uintptr        // 扩容进度：已迁移的桶索引

	extra *mapextra // 存储溢出桶等额外信息
}
```

##### 2. 桶（Bucket）结构体：`bmap`
每个 `map` 底层由多个 **桶（bucket）** 构成，每个桶可以存储多个键值对，默认是 **8 个 slots（槽位）**
```go
type bmap struct {
    tophash [8]uint8     // 每个 key 的高 8 位 hash（用于快速比较）
    // 接着是：
    // keys     [8]keytype   // 8 个 key
    // values   [8]valuetype // 8 个 value
    // overflow *bmap        // 溢出桶指针
}
```

##### 3. 查找流程（简化版）
假设我们执行：`value := myMap["apple"]`
**查找过程如下：**
1. 使用哈希函数 `h := hash("apple")` 得到哈希值。
2. 计算桶索引 `index := h & (2^B - 1)`
3. 定位到对应的桶。
4. 遍历桶中的 `tophash` 数组，快速初筛。
5. 如果找到匹配，返回对应的值。
6. 如果找不到就到溢出桶继续找。

##### 4. 扩容（resize）
Go 的 `map` 会自动进行 **扩容/缩容**，以保持性能：
- 当负载因子（元素数量 / 桶数量）过高时，**自动扩容**。
- Go 采用 **渐进式扩容**（渐进迁移旧数据到新桶），不会一次性复制全部数据。

##### 5. 哈希函数和随机化
- Go 使用自带的哈希函数（支持字符串、整数、指针等类型）。
- 使用 `hash0` 作为随机种子，每次程序运行时不同，**确保迭代顺序随机**，防止哈希冲突攻击。

##### 6. 内存结构图示意（简化）
```css
hmap
 ├── buckets → [bucket0, bucket1, bucket2, ...]
 │              ├── bmap (8对key-value + overflow指针)
 │              └── ...
 ├── oldbuckets → [旧的 buckets，扩容中使用]
 └── extra → 存储溢出桶等
```

#### map怎么判断里面有没有某个key
```go
value, exists := myMap[key]
```
示例：
```go
m := map[string]int{
	"apple":  5,
	"banana": 10,
}

v, ok := m["apple"]
if ok {
	fmt.Println("存在，值为:", v)
} else {
	fmt.Println("key 不存在")
}
```
**注意：**
- `ok` 是一个布尔值，表示 key 是否存在。
- 即使 value 是 0，也不能用 `v == 0` 判断是否存在！

#### 怎么删除某个key
使用内置的 `delete()`函数

#### map 常用操作
| 操作    | 示例代码                          |
| ----- | ----------------------------- |
| 创建    | `m := make(map[string]int)`   |
| 初始化   | `m := map[string]int{"a": 1}` |
| 访问    | `v := m["a"]`                 |
| 判断存在  | `v, ok := m["a"]`             |
| 添加/修改 | `m["b"] = 2`                  |
| 删除    | `delete(m, "a")`              |
| 遍历    | `for k, v := range m {...}`   |
| 获取长度  | `len(m)`                      |
| 清空    | `m = make(...)`               |
| 有序遍历  | `sort.Strings(keys)`          |

## 进阶
### go 的反射
Go 的反射（Reflection）是一个高级特性，允许程序在**运行时动态地检查、修改变量的类型和值**。
> Go 的反射是通过 `reflect` 包来实现的，主要有两个核心类型：
> - `reflect.Type`：表示类型信息（类似元类）
> - `reflect.Value`：表示实际的值（可以读写）

在运行时检查变量的**类型、名称、字段、方法，甚至动态修改值**。

### 如何保证并发安全？如何检测并发安全问题？
1. `Channel`：尽量避免共享数据，goroutine 之间通信，传值而非共享内存
2. `sync.Mutex`：必须共享变量时，加锁，使得多个 goroutine 修改同一个变量。
3. `sync.WaitGroup`：使用 `sync.WaitGroup`协调并发流程
4. `atomic`原子操作：高性能，适合频繁操作的计数类变量

Go 提供了内建的 **数据竞争检测工具**：
使用`-race`参数
```bash
go run -race main.go
```
或者
```bash
go test -race
```
能检测 goroutine 之间是否存在竞态条件，是并发开发必用工具
### CSP模型了解过吗
CSP（Communicating Sequential Processes，通信顺序进程）是一种并发模型，其核心思想为：
> 并发任务（进程）之间**不共享内存**，而是通过**通信**（Channel）来协作。

也就是：
- 不推荐 goroutine 直接共享变量（共享容易带来竞态）
- 推荐 goroutine **通过 channel 发送消息来协作完成任务**

### go interface概念、使用
什么是 `interface`
#### 什么是 `interface`
> **interface 是一种类型，定义了一组方法签名。** 只要某个类型实现了这些方法，它就自动实现了这个接口。

**Go 的接口是隐式实现的，不需要显式声明继承！**
#### interface 基本语法
```go
type Speaker interface {
    Speak() string
}

type Dog struct{}

func (d Dog) Speak() string {
    return "Woof"
}

func SaySomething(s Speaker) {
    fmt.Println(s.Speak())
}

func main() {
    d := Dog{}
    SaySomething(d)  // ✅ Dog 自动实现了 Speaker 接口
}
```
#### interface 的本质
**空接口 `interface{}` 底层结构：**
```go
type eface struct {
    _type *_type       // 类型信息指针
    data  unsafe.Pointer  // 数据指针
}
```

**非空接口（有方法的 interface）底层结构类似：**
```go
type iface struct {
    tab  *itab    // 方法表指针`tab`
    data unsafe.Pointer // 实际数据指针
}
```
Go 接口是一个**双指针结构**，一个指向**方法表**，一个指向**实际数据**。这也是为什么接口赋值有性能开销的原因之一。

**`_type` vs `itab`**
这两个底层结构分别用于类型信息：
- `*_type`：记录**类型元信息**（在 `eface` 中）
- `*itab`：方法表，包括 `*_type` 和**接口方法信息**（在 `iface` 中）
```go
type itab struct {
    inter  *interfaceType  // 接口的类型
    _type  *_type          // 实际值的类型
    fun    [N]uintptr      // 接口方法的函数指针
}
```

#### 空接口 `interface{}`（万能类型）
```go
func PrintAnything(x interface{}) {
    fmt.Println(x)
}
```
所有类型都实现了空接口，类似于 Python 的 `object`、Java 的 `Object`。
#### 类型断言 & 类型判断
**类型断言（Type Assertion）**
```go
var x interface{} = "hello"
str, ok := x.(string)
if ok {
    fmt.Println("It's a string:", str)
}
```

**类型 switch**
```go
switch v := x.(type) {
case int:
    fmt.Println("int:", v)
case string:
    fmt.Println("string:", v)
default:
    fmt.Println("unknown")
}
```
#### 接口的使用场景
|场景|示例|
|---|---|
|函数参数统一|`io.Reader`, `fmt.Stringer`|
|实现多态|不同类型实现同一接口|
|解耦依赖|接口隐藏实现细节，便于 mock 单元测试|
|插件化编程|各模块只需实现接口，互不依赖结构|
#### Go 标准库常用接口示例
`io.Reader`
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```
实现了这个接口的类型，都可以传给 `io.Copy()` 等函数。
#### interface 使用注意事项
|⚠️ 问题|原因|
|---|---|
|接口值为 `nil` 但实际不为 nil|接口的 data 为空但类型存在|
|空接口不能直接操作数据|必须类型断言|
|性能略低于具体类型调用|因为涉及方法表调用|
#### 总结
> **Go 的接口强调“能力”而非继承，只要你“会干这个活”，你就能当这个接口用。**




### context
#### context的用法 讲一下context
> Go 的 `context` 是用于 **控制 goroutine 生命周期** 的标准库机制。

主要用途：
- **控制取消信号**：多个 goroutine 同时取消，防止资源泄漏
- **控制超时**：超过时间自动取消操作
- **携带请求范围内数据**：在上下文中传递 trace_id、token 等

#### 服务调用其他服务出现超时该怎么办(提示用context，能够在goroutine间传递过期信息)
在项目中，你的服务调用下游服务（比如 HTTP、RPC、数据库）时，**对方服务响应慢或卡住**，你希望：
1. ⏱ 自动超时中止调用；
2. 🔁 所有子 goroutine 感知取消信号后主动退出；
3. 🧹 及时释放资源，防止协程泄露或资源阻塞。
这时，就该用 —— **`context.WithTimeout` 搭配 goroutine 使用**。
##### 实战示例：服务调用设置超时 + goroutine 共享 context
```go
package main

import (
    "context"
    "fmt"
    "net/http"
    "time"
)

func main() {
    // 创建一个 2 秒超时的 context
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel() // 确保退出时释放资源

    done := make(chan struct{})

    go func() {
        defer close(done)
        err := CallService(ctx)
        if err != nil {
            fmt.Println("服务调用失败：", err)
        } else {
            fmt.Println("服务调用成功")
        }
    }()

    // 等待子协程结束或超时
    select {
    case <-ctx.Done():
        fmt.Println("主程序：调用超时，取消任务")
    case <-done:
        fmt.Println("主程序：子协程完成")
    }
}
```

子协程中的服务调用：使用 `context` 控制超时
```go
func CallService(ctx context.Context) error {
    req, err := http.NewRequestWithContext(ctx, "GET", "http://slow-service/api", nil)
    if err != nil {
        return err
    }

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return err // 这里可能就是 context.DeadlineExceeded
    }

    defer resp.Body.Close()
    // ... 读取 body 等操作
    return nil
}
```

##### context 如何在 goroutine 间传递取消/超时信号？
- `ctx` 通过参数传给子函数或子 goroutine；
- 每个 goroutine 可以通过 `<-ctx.Done()` 感知是否“超时”或“被取消”；
- 一旦 `ctx.Done()` 被关闭，所有监听它的 goroutine 都会**立即收到通知**。
##### 总结
> 通过 `context.WithTimeout()` 创建上下文并传递给子 goroutine，所有 goroutine 可通过 `<-ctx.Done()` 统一感知“超时/取消”信号，实现服务调用的优雅超时控制与资源清理。
#### context具体有什么方法，怎么知道过期处理了，用done方法
> 在 Go 中，`context.Context` 是标准库提供的一个接口，用于**控制 goroutine 生命周期、超时、取消操作**等。主要用于在多 goroutine 之间**传递取消信号**或**超时通知**。

##### `context.Context` 提供的方法
```go
type Context interface {
    Deadline() (deadline time.Time, ok bool) // 返回 context 被取消的时间
    Done() <-chan struct{}                  // 返回一个 channel，用于检测 context 是否取消/超时
    Err() error                             // 返回取消的原因：context.Canceled or context.DeadlineExceeded
    Value(key any) any                      // 获取上下文中存储的值
}
```
**方法详解**：
1.`Done() <-chan struct{}`
> 最常用的，用于监听是否超时/取消
```go
select {
case <-ctx.Done():
    fmt.Println("操作被取消或超时")
}
```
- 返回一个只读的 channel。
- 当 context 被取消（手动 cancel 或超时）时，这个 channel 会被关闭。
- ⚠️ 一旦 `Done()` 返回的 channel 被关闭，就表示 context 被取消了。

2.`Err() error`
> 用于判断取消的具体原因
```go
err := ctx.Err()
if err != nil {
    if err == context.Canceled {
        fmt.Println("被手动取消")
    } else if err == context.DeadlineExceeded {
        fmt.Println("超时")
    }
}
```

3.`Deadline()`
> 返回 context 设置的“超时时间”
```go
deadline, ok := ctx.Deadline()
if ok {
    fmt.Println("将在", deadline, "超时")
}
```

4.`Value(key any)`
> 用于在 context 中传递值（如用户 ID、请求 ID）
```go
ctx := context.WithValue(context.Background(), "userID", 123)
val := ctx.Value("userID")
fmt.Println(val) // 123
```


##### 如何创建带超时/取消的 Context
```go
ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
defer cancel()

select {
case <-time.After(3 * time.Second):
    fmt.Println("任务完成")
case <-ctx.Done():
    fmt.Println("任务被取消：", ctx.Err())
}
```
超过 2 秒后，`ctx.Done()` 会被关闭，然后 `ctx.Err()` 返回 `context.DeadlineExceeded`。
##### 总结
| 方法           | 用途                |
| ------------ | ----------------- |
| `Done()`     | 通知 context 被取消/超时 |
| `Err()`      | 获取取消/超时的具体原因      |
| `Deadline()` | 查看何时会自动取消         |
| `Value()`    | 跨协程传递请求相关的数据      |

### GC是什么？ 什么时候会发生GC？垃圾回收过程
##### 介绍
> **GC(Garbage Collection) = 自动清理程序中不再使用的内存。**
> **GC 是 Go 自动管理内存的机制，使用三色标记算法，在程序运行时并发执行，保证低延迟，自动清理不可达对象，释放内存。**

GC 会：
- 找出“无法再被访问到”的对象（内存垃圾）
- 自动释放它们所占用的内存
- 避免手动释放内存造成的泄露、悬空指针等问题
##### 触发时机
| 触发方式   | 说明                              |
| ------ | ------------------------------- |
| 内存分配增长 | 频繁 `make/new` 分配内存，GC 会监控堆大小    |
| 定时触发   | GC 默认每隔两分钟会强制触发一次               |
| 手动调用   | `runtime.GC()` 可以强制触发一次         |
| 环境变量调节 | 通过 `GOGC` 设置 GC 触发频率（默认是 `100`） |

##### GC 过程（三色标记法）
Go 的 GC 主要分为3个阶段：
1. **标记阶段（Mark）**
	- 从根对象（全局变量、栈上的变量）出发，**遍历可达对象**，打上标记
	- 使用三色标记法：白（待清除）、灰（正在访问）、黑（已标记）
2. **清扫阶段（Sweep）**
	- 回收所有“白色对象”（不可达、无引用）
3. **压缩阶段（Go 不需要）**
	- Go 使用指针，所以不需要“移动对象压缩堆”这一步（不像 Java）

**详细流程**：
**第一步**：每次新创建的对象，默认的颜色都是标记为“白色”，如图所示。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250717110755807.png)
所谓“程序”，则是一些对象的根节点集合。所以我们如果将“程序”展开，会得到类似如下的表现形式，如图所示。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250717110604747.png)

**第二步**：每次 GC 回收开始，从根节点开始遍历所有对象，把遍历到的对象从白色集合放入“灰色”集合
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250717110904533.png)

**第三步**：遍历灰色集合，将灰色对象引用的对象从白色集合放入灰色集合，之后将此灰色对象放入黑色集合
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250717111037336.png)这一次遍历是只扫描灰色对象，将灰色对象的第一层遍历可抵达的对象由白色变为灰色

**第四步**, 重复**第三步**, 直到灰色中无任何对象，如图所示。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250717111133234.png)
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250717111144171.png)
当我们全部的可达对象都遍历完后，灰色标记表将不再存在灰色对象，目前全部内存的数据只有两种颜色，黑色和白色。那么**黑色对象**就是我们**程序逻辑可达（需要的）对象**，这些数据是目前支撑程序正常业务运行的，是合法的有用数据，不可删除，**白色的对象**是**全部不可达对象**，目前程序逻辑并不依赖他们，那么白色对象就是内存中目前的垃圾数据，需要被清除。

**第五步**: 回收所有的白色标记表的对象. 也就是回收垃圾，如图所示。
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250717111222702.png)

### 协程
#### 协程是什么？
协程（Goroutine）是 Go 语言中最核心的并发原语之一，可以把它理解为：
**轻量级的线程**，由 Go 运行时（runtime）调度和管理，而不是由操作系统直接调度。

特点：
- **轻量级**：一个 Go 程序可以轻松开启数十万甚至更多的 goroutine
- **非抢占式调度**：Go runtime 在合适点自动切换执行
- **自动管理栈空间**：初始栈很小（2KB），按需自动扩容
- **高并发**：非常适合 I/O 密集型和并发任务
- **与通道配合**：通常配合 `channel` 实现通信（即 CSP 模型）
#### 进程线程和协程的区别
##### 基本概念
**进程（Process）**：程序运行的最小单位，是操作系统分配资源的基本单位。每个进程拥有独立的内存空间。
**线程（Thread）**：操作系统调度的基本单位，是进程内部的执行流。多个线程共享进程的资源（如内存、文件等）。
**协程（Coroutine）**：用户态的“轻量线程”，由程序/语言自身调度（如 Go 中的 `goroutine`），开销远低于线程。
##### 调度机制
| 对象  | 由谁调度               | 特点               |
| --- | ------------------ | ---------------- |
| 进程  | 操作系统               | 严格隔离、安全          |
| 线程  | 操作系统               | 支持并发，但有共享风险      |
| 协程  | 语言/库（如 Go runtime） | 用户态调度，无需系统干预，更高效 |
##### Go 中实践
| 实体  | Go 中的体现          |
| --- | ---------------- |
| 进程  | `main` 程序进程      |
| 线程  | Go runtime 管理的 M |
| 协程  | `goroutine`      |
#### goroutine 和系统线程的区别
| 实体  | Go 中的体现          |
| --- | ---------------- |
| 进程  | `main` 程序进程      |
| 线程  | Go runtime 管理的 M |
| 协程  | `goroutine`      |

#### Go中GMP调度原理   介绍一下GMP？M和P的数量是怎么指定的？能不能去掉p层？
##### 介绍：
GMP 模型是 Go 协程调度系统：
**G(Goroutine):** **每个 goroutine 是一个待调度的任务**
**M(Machine):** **OS 线程（实际执行任务的实体）绑定 P 来实际执行 goroutine**
**P(Processor):** **调度器，执行上下文 + 本地队列，负责调度 G 给 M**

GMP 模型执行流程如下图所示：
![g.m.p.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250705163144636.png)

##### 常见问题：
**1. `go runtime` 是什么**
 `runtime` 是 **Go 自带的微型操作系统**，**连接 Go 程序与操作系统之间的桥梁**，它让 goroutine、channel 等高级功能能高效运行而不用依赖 OS 的重型线程模型，提供了**协程调度、内存管理、GC（垃圾回收）、栈扩容、系统调用、反射、panic/recover 等底层机制**
 
**2.局部队列和 P 的数量是怎么确定的？**
P 的数量是由 `GOMAXPROCS`决定的
- 启动时，Go runtime 会根据 `GOMAXPROCS` 的值分配 P 的数量。
- 默认值是 **当前机器的 CPU 核数**，你可以通过如下方式控制：
```go
runtime.GOMAXPROCS(n) // 设置最大可并行执行 P 的数量
```
所以：**P 的数量是固定的，等于 GOMAXPROCS，和局部队列数量一一对应**。

每个 P 会维护一个“局部队列”（local runq）
- 该队列用于保存要运行的 G（goroutine）。
- 每个 P 的 local queue 最多能容纳 **256 个 G**。

**3.既然有局部队列，为什么还要设计出全局队列？**
这是 Go 调度器为了实现**负载均衡 + 防止局部拥堵**而设计的机制

**4.G 被放入全局队列的几种情况：**

| 情况                       | 说明                             |
| ------------------------ | ------------------------------ |
| 1️⃣ P 的局部队列满了（超过 256）    | 多余的 G 会被放入全局队列                 |
| 2️⃣ 当前没有可用 P（比如刚开始程序执行时） | G 会暂时放入全局队列等待被调度               |
| 3️⃣ 调度器偷任务失败             | P 会从全局队列尝试获取 G                 |
| 4️⃣ 系统调用返回、唤醒 G 时        | 有概率被放到全局队列，而不是 P 的本地队列（随机分散负载） |

**5.local runq 和 global runq 的使用顺序优先级**
>**调度器优先使用局部队列，其次是用全局队列，最后才从其他 P 偷**。

优先级顺序如下：
1. 当前 P 的 local runq
2. 从 global runq 获取 G
3. 偷取其他 P 的 G（work stealing）
**6.work stealing（任务窃取）**
work stealing（任务窃取）是一种并发调用策略：
>**当一个 P（Processor）自己的任务队列空了，它会尝试从其他 P 的任务队列中“偷” goroutine 来执行，实现 goroutine 的全局负载均衡。**

目的是：
- **保持调度器的活跃性**
- **防止某些 P 空转，其他 P 忙不过来**
- **实现动态负载均衡**

##### M和P的数量是怎么指定的？
**P 的数量：**
- **固定值**，由 `GOMAXPROCS` 决定
- 默认值 = CPU 核数
- 一旦设置，**运行中不能自动增长或缩减**
- 每个 P 有自己的本地 G 队列
**M 的数量：**
- M 是 **操作系统线程（OS thread）**
- **动态分配、按需增加，无上限（理论值）**
- 只要有 goroutine 需要执行且找不到空闲线程，Go runtime 就会启动新的 M
- 也就是说：**M 数量 ≥ 活跃的 P 数量**，有时远多于 P

##### 能不能去掉p层？
> 不能去掉 P。P 是 Go 并发模型的“**调度核心**”，**承担了调度器的角色**，调度逻辑的核心都围绕它展开。

**P 的功能总结：**

| 功能                  | 说明                                   |
| ------------------- | ------------------------------------ |
| 📦 管理 G 队列          | 每个 P 维护 local runq                   |
| ⚙️ 控制并发数            | 由 GOMAXPROCS 决定，限制同时运行 goroutine 的上限 |
| 🔁 调度 goroutine     | P 是 goroutine 的调度中枢                  |
| 🔄 与 M 解绑           | M 阻塞时，P 可绑定其他 M 保持调度活跃               |
| 🔄 Work stealing 单元 | P 可从其他 P 偷任务，帮助负载均衡                  |

#### 协程阻塞的原因有哪些?
| 编号  | 原因类别                  | 举例说明      |
| --- | --------------------- | --------- |
| 1️⃣ | 通道（channel）操作阻塞       | 发送或接收阻塞   |
| 2️⃣ | `select` 所有 case 都阻塞  | 没有默认分支    |
| 3️⃣ | `sync.Mutex` 竞争锁阻塞    | 被其他协程持有   |
| 4️⃣ | `WaitGroup.Wait()` 阻塞 | 等待未完成的协程  |
| 5️⃣ | `time.Sleep`、I/O 等    | 主动阻塞或外部阻塞 |
| 6️⃣ | goroutine 自身死循环       | 一直占用 CPU  |

#### 协程什么情况下会退出
1. **函数正常执行结束**
2. **函数内触发`return`语句**
3. **发生 `panic` 且没有 `recover()`，会崩掉这个协程**
4. **阻塞资源关闭 / channel 关闭**
	如果 goroutine 在等待 channel / I/O，且资源关闭了，也会退出。
5. **被 `context` 主动取消（推荐用于可控退出）**
6. **主协程结束，程序退出，所有子协程强制终止**

#### 如何实现协程池
##### 协程池是什么？
> 协程池是一种**限制并发 goroutine 数量**的机制，它通过**复用固定数量的工作 goroutine 来处理大量任务**。
##### 实现思路（通用模板）
1. 创建一个带缓冲的任务队列 `jobs`
2. 创建 `N` 个工作 goroutine，从任务队列中取任务执行
3. 主线程向任务队列发送任务
4. 使用 `sync.WaitGroup` 等待所有任务完成

#### go语言中的主进程被杀掉后，剩下的线程会怎么办？怎么回收
> 如果主进程（主 goroutine）**结束或被杀掉**，那么 **整个 Go 程序会立即终止**，包括所有正在运行的 goroutine —— **不会继续运行、也不会自动清理回收**，**操作系统会强制杀掉整个进程，包括所有线程/协程**。

**详细解释：**
Go 程序中：
- **goroutine 并不是独立于进程存在的**，而是属于 Go 运行时（runtime）调度的、**依附于进程内的线程（M）运行的协程（G）**。
- 当主进程或主 goroutine 执行完毕或被外部 `kill` 掉：
    - 所有协程（goroutine）将**立刻全部停止**。
    - 没有清理过程，**内存、文件句柄、网络连接等全部被操作系统强制回收**。

#### go的竞争条件 Mutex
当多个 goroutine 同时访问/修改同一个共享变量，**且至少有一个是写操作**，就会出现 **竞态条件**。
**检测竞态条件**
Go 提供了检测工具：
```bash
go run -race main.go
```
如果由竞态，终端会打印：
```bash
WARNING: DATA RACE
```

`Mutex`
Go 提供了 `sync.Mutex` 互斥锁，保证**同一时间只有一个 goroutine 能访问共享资源**。

#### 100个协程执行了50个，51panic后面的还执行吗 - 不想退出怎么做
不会影响其他协程继续执行。
Go 的 goroutine 是独立的，**一个协程 panic，不会直接导致其他 goroutine 崩溃**，但如果 panic **没有被恢复（recover）**，**会导致主协程(main 函数)崩溃，进而整个程序崩溃！**

**panic 的传播机制：**
- panic 是线程级别的
- **但若 panic 没有在协程中被 `recover()` 捕获**，会一直冒泡到运行时（runtime），最终打印错误并终止整个进程！
##### 正确做法：
**每个 goroutine 内部捕获 panic，防止崩主程序**
```go
for i := 0; i < 100; i++ {
    go func(i int) {
        defer func() {
            if r := recover(); r != nil {
                fmt.Printf("协程 %d 发生 panic: %v\n", i, r)
            }
        }()
        doWork(i)
    }(i)
}
```

#### go 发现一个运行时间过长的协程，如何感知并停止。拆成“如何发现协程泄露”和“如何控制协程在某个时机退出”两个问题
##### 如何发现 goroutine 泄露？
**什么是 goroutine 泄露？**
> 程序运行中产生的 goroutine **一直未退出，也没有必要继续运行**，它们可能因**阻塞**、**死锁**、**没被取消**等原因“悬挂”在后台，造成资源浪费甚至系统崩溃。

**检测方式**：
**1.使用运行时打印 goroutine 数量**
```go
import "runtime"

fmt.Println("Current goroutine count:", runtime.NumGoroutine())
```
- **持续增长不下降**：可能有泄露
- 可搭配 Prometheus 监控 `NumGoroutine()` 值
**2.使用 `pprof` 工具分析协程状态**
内置 `net/http/pprof` 可查看 goroutine 堆栈：
```go
import _ "net/http/pprof"
import "net/http"

go func() {
    http.ListenAndServe("localhost:6060", nil)
}()
```
然后访问：
```bash
http://localhost:6060/debug/pprof/goroutine?debug=2
```
查看是否有大量 goroutine 停在某个函数（比如 `<-ch` 处），判断是否泄露。

**3.常见泄露场景**

| 场景            | 示例                 |
| ------------- | ------------------ |
| 阻塞在 channel 上 | `<-ch` 没有发送方       |
| 没有退出的 select  | select 中没有触发分支     |
| 死锁            | 多个协程等待彼此释放         |
| 外部超时/中断后仍运行   | HTTP/DB 请求超时，协程未终止 |
##### 如何控制 goroutine 在某个时机退出？
**1. 使用 `context.Context` 控制协程退出（推荐 ✅）**
```go
import (
    "context"
    "time"
)

func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("Goroutine exit")
            return
        default:
            fmt.Println("Working...")
            time.Sleep(1 * time.Second)
        }
    }
}

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()

    go worker(ctx)

    time.Sleep(5 * time.Second)
    fmt.Println("Main done")
}
```
`context` 可传入 `timeout`、`cancel`、`deadline`，是协程退出的最佳实践。

**2. 使用 channel 控制退出**
```go
func worker(stop chan struct{}) {
    for {
        select {
        case <-stop:
            fmt.Println("Goroutine exit")
            return
        default:
            fmt.Println("Working...")
            time.Sleep(time.Second)
        }
    }
}

func main() {
    stop := make(chan struct{})
    go worker(stop)

    time.Sleep(3 * time.Second)
    close(stop)  // 通知退出
}
```
适合简单协程控制，但不如 `context` 易组合和可控。

**3. 使用 `WaitGroup` 等待所有协程完成**
```go
var wg sync.WaitGroup

func worker() {
    defer wg.Done()
    fmt.Println("Working...")
    time.Sleep(2 * time.Second)
}

func main() {
    wg.Add(1)
    go worker()
    wg.Wait()
    fmt.Println("All done")
}
```
只能“等待”退出，**不能主动终止协程**。

#### 子goroutine发生panic会影响父goroutine吗
| 问题                                | 是否会发生                        |
| --------------------------------- | ---------------------------- |
| 子 goroutine panic 是否影响父 goroutine | ❌ 不会传播，只崩自己                  |
| panic 是否自动退出程序                    | ❌ 不会，除非是 main 协程 panic       |
| 如何安全使用 goroutine                  | ✅ `defer + recover` 捕获 panic |
| 如何汇报错误到主 goroutine                | ✅ 用 channel 通信上报             |
#### 在父子 goroutine 中 defer 一般在哪调用
> 在 Go 中，无论是 **父 goroutine** 还是 **子 goroutine**，`defer` 的作用域和执行时机是非常明确的——**它只会在当前 goroutine 中函数返回时触发执行**，不会跨 goroutine 调用。

##### 使用示例
正确使用方式：**defer 写在 goroutine 函数体内**
```go
func main() {
    go func() {
        defer fmt.Println("子协程 defer 被执行")
        fmt.Println("子协程开始")
    }()

    defer fmt.Println("主协程 defer 被执行")
    fmt.Println("主协程开始")

    time.Sleep(1 * time.Second)
}
```
**输出（顺序不一定）：**
```go
主协程开始
子协程开始
子协程 defer 被执行
主协程 defer 被执行
```
每个 goroutine 只管自己的 defer，**不会触发其他协程的 defer 执行**。

##### 总结
| 结论                           | 说明                          |
| ---------------------------- | --------------------------- |
| `defer` 是 goroutine 局部的      | 只能作用于定义它的函数体内               |
| 父 goroutine 的 `defer` 不会等子协程 | 主程序会在自己的逻辑跑完后触发 defer，不等子协程 |
| 子协程的清理必须在子协程中写 `defer`       | 否则资源无法释放                    |
| 需要等待子协程时用 `WaitGroup`        | 保证主协程不会提前退出                 |

### 新特性
#### 有用过go的新特性吗？说一下项目中怎么用的
泛型（1.18引入）









