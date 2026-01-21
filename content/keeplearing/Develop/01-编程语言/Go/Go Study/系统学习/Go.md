## 变量
## 数据类型
### 字符串
#### 1. print 格式化输出类型：
- `%c`：单一字符
- `%T`：动态类型
- `%v`：本来值的输出
- `%+v`：字段名+值打印
- `%d`：十进制打印数字
- `%p`：指针，十六进制
- `%f`：浮点数
- `%b`：二进制
- `%s`：string

#### 2. 字符串查找方法
`strings.Index(s, substr)`：返回`substr`在`s`中首次出现的位置（索引），找不到返回 -1
`strings.LastIndex(s, substr)`：返回`substr`在`s`中最后一次出现的位置


### 指针 
指针（pointer）在Go语言中可以被拆分为两个核心概念：
- **类型指针**，允许 对这个指针类型的数据进行修改，传递数据可以直接使用指针，而无须拷贝数据，类型指针不能进行偏移和运算。
- **切片**，由指向起始元素的原始指针、元素数量和容量组成。

#### 1. 如何理解指针
**变量、指针和地址三者的关系是，每个变量都拥有地址，指针的值就是地址**
> 当使用`&` 操作符对普通变量进行取地址操作并得到变量的指针后，可以对指针使用`*` 操作符，也就是指针取值

取地址操作符`&` 和取值操作符`*` 是一对互补操作符，`&` 取出地址，`*` 根据地址取出地址指向的值

变量、指针地址、指针变量、取地址、取值的相互关系和特性如下：
- 对变量进行取地址操作使用`&` 操作符，可以获得这个变量的指针变量。
- 指针变量的值是指针地址。
- 对指针变量进行取值操作使用`*` 操作符，可以获得指针变量指向的原变量的值。

#### 2. 创建指针的另一种方法
**new() 函数可以创建一个对应类型的指针，创建过程会分配内存，被创建的指针指向默认值。**

### Map
`sync.Map` 有以下特性：
- 无须初始化，直接声明即可。
- `sync.Map` 不能使用 map 的方式进行取值和设置等操作，而是使用 `sync.Map` 的方法进行调用，`Store` 表示存储，`Load` 表示获取，`Delete` 表示删除。
- 使用 `Range` 配合一个回调函数进行遍历操作，通过回调函数返回内部遍历出来的值，`Range` 参数中回调函数的返回值在需要继续迭代遍历时，返回 true，终止迭代遍历时，返回 false
```go
func main(){
	var scene sync.Map
	// 将键值对保存到 sync.Map
	// sync.Map 将键和值以 interface{} 类型进行保存
	scene.Store("A", 97)
	scene.Store("B", 100)
	scene.Store("C", 200)
	// 从 sync.Map 中根据键取值
	fmt.Println(scene.Load("B"))
	// 根据键删除对应的键值对
	scene.Delete("B")
	// 遍历所有 sync.Map 中的键值对
	// 遍历需要提供一个匿名函数，参数为 k、v，类型为 interface{}，每次 Range() 在遍历一个元素时，都会调用这个匿名函数把结果返回
	scene.Range(func(k, v interfaceP{}) bool{
		fmt.Println("iterate", k, v)
		return true
	})
}
```

**`sync.Map` 为了保证并发安全有一些性能损失，因此在非并发情况下，使用 map 相比使用 `sync.Map` 会有更好的性能。**

### new 和 make
`make` 关键字的主要作用是创建 `slice`、`map` 和 `Channel` 等内置的数据结构，而 `new` 的主要作用是为类型申请一片内存空间，并返回指向这片内存的指针。
1. `make` 分配空间后，会进行初始化，`new` 分配的空间被清零
2. `new` 分配返回的是指针，即类型 `*Type`。`make` 返回引用，即 `Type`
3. `new` 可以分配任意类型的数据

### 闭包
> 闭包，指的是一个拥有许多变量和绑定了这些变量的环境的表达式（通常是一个函数），因而这些变量也是该表达式的一部分。
> 闭包 = 函数 + 引用环境


### 延迟调用
> Go 语言的 defer 语句会将其后面跟随的语句进行延迟处理

**defer 特性**：
1. 关键字 defer 用于注册延迟调用
2. 这些调用知道 return 前才被执行，因此，可以用来做资源清理
3. 多个 defer 语句，按先进后出的方式执行
4. defer 语句中的变量，在 defer 声明时就决定了


### 结构体
**结构体的定义只是一种内存布局的描述，只有当结构体实例化时，才会真正地分配内存**
#### 实例化
##### 取结构体的地址实例化：
在 Go 语言中，对结构体进行`&` 取地址操作时，视为对该类型进行一次 new 的实例化操作，取地址格式如下：
`ins := &T{}`
其中：
- T 表示结构体类型。
- ins 为结构体的实例，类型为 `*T`，是指针类型。

#### 匿名结构体
匿名结构体没有类型名称，无须通过 type 关键字定义即可以直接使用。


### 方法
#### 接收器
接收器的格式如下：
```go
func (接收器变量 接收器类型) 方法名(参数列表) (返回参数) {
	函数体
}
```
- **接收器变量**：接收器中的参数变量名在命名时，官方建议使用接收器类型名的第一个小写字母，而不是 self、this 之类的命名。例如，Socket 类型的接收器变量应该命名为 s，Connector 类型的接收器变量应该命名为 c 等。
- **接收器类型**：接受器类型和参数类似，可以是**指针类型**和**非指针类型**。
- **方法名、参数列表、返回参数**：格式与函数定义一致。

##### 指针类型的接收器：
指针类型的接收器由一个结构体的指针组成，更接近于面向对象中的 this 或者 self。
由于指针的特性，调用方法时，**修改接收器指针的任意成员变量，在方法结束后，修改都是有效的**。

##### 非指针类型的接收器：
当方法作用于非指针接收器时，Go 语言会在代码运行时将接收器的值复制一份，在非指针接收器的方法中可以获取接收器的成员值，但**修改后无效**。

### 接口
接口（interface）是一种类型

#### 空接口
空接口是指没有定义任何方法的接口。
因此任何类型都实现了空接口。
空接口类型的变量可以存储任意类型的变量。

##### 空接口应用
1. **空接口作为函数的参数**
	使用空接口实现可以接收任意类型的函数参数。
```go
// 空接口作为函数参数
func show(a interface{}){
	fmt.Printf("type:%T value:%v\n", a, a)
}
```

2. **空接口作为 map 的值**
	使用空接口实现可以保存任意值的字典。
```go
// 空接口作为 map 值
var studentInfo = make(map[string]interface{})
studentInfo["name"] = "李白"
studentInfo["age"] = 18
studentInfo["married"] = false
fmt.Println(studentInfo)
```

##### 类型断言
**接口值**
一个接口的值（简称接口值）是由**接口的动态类型**和**动态值**两部分组成的。
想要判断空接口中的值可以使用类型断言，其语法格式：
`x.(T)`
其中：
1. x：表示类型为 `interface{}` 的变量
2. T：表示断言 x 可能是的类型。

该语法返回两个参数，第一个参数是 x 转化为 T 类型后的变量，第二个值是一个布尔值，若为 true 则表示断言成功，为 false 则表示断言失败。

### Goroutine
示例
```go
package main  
  
import (  
    "fmt"  
    "sync"    "time")  
  
var students = map[int64]*student{  
    1: {  
       id:      1,  
       name:    "nike",  
       age:     18,  
       address: "xxx1号",  
    },  
    2: {  
       id:      2,  
       name:    "curo",  
       age:     19,  
       address: "xxx2号",  
    },  
}  
  
type student struct {  
    id      int64  
    name    string  
    age     int  
    address string  
}  
  
func getStudentByID(id int64) (*student, error) {  
    // 网络请求， DB Redis    time.Sleep(time.Millisecond * 100)  
    return students[id], nil  
}  
  
func main() {  
    start := time.Now()  
    ids := []int64{1, 2}  
    idToAgeMapper := make(map[int64]int)  
  
    var (  
       eg    sync.WaitGroup  
       mutex sync.Mutex  
    )  
  
    for _, id := range ids {  
       eg.Add(1)  
       go func(id int64) {  
          defer func() {  
             mutex.Unlock()  
             eg.Done()  
          }()  
          s, err := getStudentByID(id)  
          if err != nil {  
             return  
          }  
          mutex.Lock()  
          idToAgeMapper[s.id] = s.age  
          fmt.Printf("id = [%d], s = [%+v]\n", id, s)  
       }(id)  
    }  
  
    eg.Wait()  
    end := time.Since(start)  
    fmt.Println(end)  
}
```



### Channel
#### 单向通道
限制通道在函数中只能发送或只能接收。
示例：
```go
func counter(out chan<- int) {
    for i := 0; i < 100; i++ {
        out <- i
    }
    close(out)
}

func squarer(out chan<- int, in <-chan int) {
    for i := range in {
        out <- i * i
    }
    close(out)
}
func printer(in <-chan int) {
    for i := range in {
        fmt.Println(i)
    }
}

func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    go counter(ch1)
    go squarer(ch2, ch1)
    printer(ch2)
}
```
- `chan<- int` 是一个只能发送的通道，可以发送但是不能接收；
- `<-chan int` 是一个只能接收的通道，可以接收但是不能发送。
在函数传参及任何赋值操作中将双向通道转换为单向通道是可以的，但反过来是不可以的。

#### 无缓冲channel单向传递
```go
package main  
  
import (  
    "fmt"  
    "sync"    "time")  
  
func send(ch chan<- int) {  
    for i := 0; i < 3; i++ {  
       ch <- i  
       fmt.Println("发送数据:", i)  
    }  
    close(ch)  
}  
  
func receive(ch <-chan int, wg *sync.WaitGroup) {  
    defer wg.Done()  
    for data := range ch {  
       time.Sleep(time.Second * 1)  
       fmt.Println("接收数据：", data)  
    }  
}  
  
func main() {  
    var wg sync.WaitGroup  
  
    ch := make(chan int)  
    wg.Add(1)  
    go send(ch)  
    go receive(ch, &wg)  
    wg.Wait()  
}
```
无缓冲的 channel 相当于同步传输
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250723142402329.png)
程序运行时，发一个数据马上接收一个数据

#### 有缓冲channel异步传输
```go
package main  
  
import (  
    "fmt"  
    "sync"    "time")  
  
func send(ch chan<- int) {  
    for i := 0; i < 3; i++ {  
       ch <- i  
    }  
    fmt.Println("发送结束")  
    close(ch)  
}  
  
func receive(ch <-chan int, wg *sync.WaitGroup) {  
    defer wg.Done()  
    for data := range ch {  
       time.Sleep(time.Second * 1)  
       fmt.Println("接收数据：", data)  
    }  
}  
  
func main() {  
    var wg sync.WaitGroup  
  
    ch := make(chan int, 3)  
    wg.Add(1)  
    go send(ch)  
    go receive(ch, &wg)  
    wg.Wait()  
}
```
带缓冲channel即可实现异步传输：
![image.png](https://tyrese-1317134930.cos.ap-shanghai.myqcloud.com/imgs/blog/20250723142105457.png)
程序刚开始运行时，数据就全部发送完毕，然后才开始接收数据

### select
`select` 语句用于监听多个 channel 上的消息。当有一个 channel 有数据可读或可写时，就会执行对应的 case 分支
- 避免无缓冲 channel 死锁
- 监听多个 channel

`select`的使用类似于`switch`语句，它有一系列`case`分支和一个默认的分支。每个`case`会对应一个通道的通信（接收或发送）过程。`select`会一直等待，直到某个`case`的通信操作完成时，就会执行`case`分支对应的语句。具体格式如下：
```go
select {
    case <-chan1:
       // 如果chan1成功读到数据，则进行该case处理语句
    case chan2 <- 1:
       // 如果成功向chan2写入数据，则进行该case处理语句
    default:
       // 如果上面都没有成功，则进入default处理流程
    }
```

**select可以同时监听一个或多个channel，直到其中一个channel ready**
```go
package main

import (
   "fmt"
   "time"
)

func test1(ch chan string) {
   time.Sleep(time.Second * 5)
   ch <- "test1"
}
func test2(ch chan string) {
   time.Sleep(time.Second * 2)
   ch <- "test2"
}

func main() {
   // 2个管道
   output1 := make(chan string)
   output2 := make(chan string)
   // 跑2个子协程，写数据
   go test1(output1)
   go test2(output2)
   // 用select监控
   select {
   case s1 := <-output1:
      fmt.Println("s1=", s1)
   case s2 := <-output2:
      fmt.Println("s2=", s2)
   }
}
```

**如果多个channel同时ready，则随机选择一个执行**
```go
package main

import (
	"fmt"
)

func main() {
	// 创建2个管道
	intChan := make(chan int, 1)
	stringChan := make(chan string, 1)
	go func() {
		//time.Sleep(2 * time.Second)
		intChan <- 1
	}()
	go func() {
		stringChan <- "hello"
	}()
	select {
	case value := <-intChan:
		fmt.Println("int:", value)
	case value := <-stringChan:
		fmt.Println("string:", value)
	}
	fmt.Println("main结束")
}
```

**可以用于判断管道是否存满**
```go
package main

import (
   "fmt"
   "time"
)

// 判断管道有没有存满
func main() {
   // 创建管道
   output1 := make(chan string, 10)
   // 子协程写数据
   go write(output1)
   // 取数据
   for s := range output1 {
      fmt.Println("res:", s)
      time.Sleep(time.Second)
   }
}

func write(ch chan string) {
   for {
      select {
      // 写数据
      case ch <- "hello":
         fmt.Println("write hello")
      default:
         fmt.Println("channel full")
      }
      time.Sleep(time.Millisecond * 500)
   }
}
```








