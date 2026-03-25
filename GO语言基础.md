## GO语言基础

### 基本语法、包管理、go mod

#### Go 的基本语法核心 

Go 的语法以“简洁”著称，没有太多语法糖，但有几个特性是面试官特别爱问的“坑”。

**变量声明与“短变量声明” (`:=`)**

Go 有多种声明方式，但 `:=` 最常用也最容易出错。

- **标准声明:** `var name string = "Gemini"` (繁琐，通常用于全局变量)
- **类型推导:** `var name = "Gemini"`
- **短变量声明:** `name := "Gemini"` (最常用，**只能在函数内部使用**)

**可见性规则 (Public/Private)**

Go 没有 `public`, `private` 关键字，而是通过**首字母大小写**来控制。

- **首字母大写:** 公开 (Public/Exported)。其他包可以访问。
	- 例如: `func CreateUser()`, `type User struct`
- **首字母小写:** 私有 (Private/Unexported)。只有当前包 (`package`) 内部可见。
	- 例如: `func internalHelper()`, `var connectionString`

**函数的多返回值**

这是 Go 的一大特色，主要用于**错误处理**。

```go
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("cannot divide by zero")
    }
    return a / b, nil
}
```

**只有一种循环**

Go 没有 `while` 或 `do-while`，只有 `for`。

```go
// 相当于 while(true)
for {
    // 逻辑
}

// 标准 for
for i := 0; i < 10; i++ {
    // 逻辑
}
```

#### 包管理与 Go Modules (`go mod`)

在 Go 1.11 之前，包管理是 Go 的痛点（GOPATH 时代）。现在，**Go Modules** 是官方推荐的、唯一的标准。

什么是 Go Modules?

它是一个依赖管理系统，允许你在 **GOPATH** 之外的任何目录下创建项目，并且明确记录每个依赖包的版本。

核心文件

在你的项目根目录下，通常有两个文件：

- **`go.mod` (必须):** 相当于 Java 的 `pom.xml` 或 Node 的 `package.json`。
	- 定义了模块名 (`module github.com/yourname/project`)。
	- 定义了 Go 的版本。
	- 记录了直接依赖包的版本 (`require ...`)。
- **`go.sum` (必须):** 这是一个校验文件。
	- 记录了依赖包的哈希值 (Checksum)。
	- **作用:** 保证你和同事下载的依赖包内容是完全一致的，防止依赖包被篡改。

 

**常用命令**

**`go mod init <模块名>`**: 初始化一个新项目。

- 例如: `go mod init github.com/zhangsan/my-server`

**`go mod tidy`**: **最重要的命令**。

- 自动**添加**你代码中引用了但 `go.mod` 没记录的依赖。
- 自动**删除** `go.mod` 中记录了但你代码没用到的依赖。
- 保持依赖树干净。

**`go get <包名>`**: 下载并安装依赖。

- 例如: `go get -u github.com/gin-gonic/gin` ( `-u` 表示更新到最新版)

### 数据类型、切片/数组/Map、Struct、接口 Interface

#### 数据类型

##### 1. String 的底层结构

在 Go 中，`string` 是不可变的。它的底层结构非常像切片（Slice），但没有 `Capacity`。

它占用 **16 字节**（64位机器上）：

- `Data` (8字节): 指向底层字节数组的指针。
- `Len` (8字节): 字符串的字节长度。

 面试黑科技：String 和 []byte 的零拷贝转换

通常 `string(bytes)` 或 `[]byte(str)` 会发生内存拷贝。但在高性能场景（如网关、高频序列化）下，我们需要“零拷贝”转换。

**面试官问：** *“如何不发生内存拷贝，将 string 转为 []byte？”*

**答案（使用 `unsafe` 包）：**

```go
import(
	"unsafe"
	"fmt"
)
func main(){
	s:="hello"
	// 利用 unsafe.StringData 获取底层指针
	// 利用 unsafe.Slice 构造切片，长度为 len(s)
	b:=unsafe.Slice(unsafe.StringDate(s),len(s))
	fmt.Printf("%t\n", s == string(b)) // true，但 b 和 s 共享底层内存
    // ⚠️ 注意：千万不要修改 b！因为 s 是不可变的，修改 b 会导致不可预知的崩溃或行为。
}
```

##### **数组 (Array) vs 切片 (Slice)**

核心区别

**数组:** **值类型**。长度是类型的一部分（`[3]int` 和 `[4]int` 是不同类型）。赋值或传参会**拷贝整个数组**（很慢）。

**切片:** **引用类型**（轻量级）。底层是一个结构体，只包含三个字段：

1. `ptr`: 指向底层数组某个元素的指针。
2. `len`: 当前长度。
3. `cap`: 容量（从 `ptr` 开始到底层数组末尾的长度）。

切片的扩容机制 (Grow)

当 `append` 超过 `cap` 时，切片会扩容：

- **Go 1.18 之前:** 小于 1024 双倍扩容，大于 1024 扩 1.25 倍。
- **Go 1.18 之后:** 算法更平滑，但大致逻辑依然是：小切片翻倍，大切片缓慢增加。

面试必考题：**切片截取导致内存泄漏**

**场景：** 你读取了一个 100MB 的大文件存入 `bigSlice`，然后只想要前 10 个字节 `smallSlice := bigSlice[:10]`。

**问题：** 只要 `smallSlice` 还在内存中，那 100MB 的底层数组就**无法被回收**，因为 `smallSlice` 依然持有对它的引用。

**解决：** 使用 `copy` 复制你需要的那部分数据到新的切片中。

面试必考题：**Nil Slice vs Empty Slice**

```go
var s1 []int         // Nil Slice (s1 == nil) -> 推荐使用
s2 := make([]int, 0) // Empty Slice (s2 != nil)
s3 := []int{}        // Empty Slice (s3 != nil)
```

- **区别：** JSON 序列化时，`nil` 切片变成 `null`，空切片变成 `[]`。这点在写 API 接口时非常重要！



#### 切片 slice

slice 是一个“描述符”：**指针 + len + cap**（底层指向数组）

`append` 可能触发扩容，扩容后会指向新数组

- slice **传参是值拷贝**，但拷贝的是描述符，底层数组可能共享

```go
a := []int{1,2,3}
b := a[:2]        // b 共享 a 的底层数组
b[0] = 100
fmt.Println(a)    // [100 2 3]
```



1.**slice的底层结构是怎样的？**

Go 的 slice 底层不是数组本身，而是一个 **slice header**，由三部分组成：

1）**指针 Data/ptr**：**指向底层数组中 slice 起始元素的位置**；
 2）**len**：当前 slice 的长度；
 3）**cap**：从这个起始位置到底层数组末尾的容量。

slice 的数据存放在它指向的**底层数组**里，所以 slice 变量之间赋值或传参只会复制 header，多个 slice 可能共享同一个底层数组；

对元素的修改会互相影响。

`append` 在 `len < cap` 时通常复用原数组，在 `len == cap` 时会扩容分配新数组并拷贝数据，之后就不再共享。



2.**slice和int float有什么区别?** 

`int/float` 是**基本数值类型**，变量里存的就是数值本身，大小固定（`int` 依赖架构、`float32/64` 固定），**值拷贝**时复制的是数值。

`slice` 不是基本类型，它是一个**切片描述符**（`ptr + len + cap`），本身不直接存数据，数据在它指向的**底层数组**里；

拷贝/传参复制的是这 3 个字段，多个 slice 可能**共享同一底层数组**，修改元素会互相影响；

并且 slice 可能是 `nil`，`append` 还可能触发扩容换底层数组。



3.**如果在slice里面存放指针,对slice进行复制,修改新的某个下标变量会影响老的吗?**

取决于修改的方式

**改 slice 元素本身（指针值）：**

- 如果你只是 `s2 := s1` 这种复制，两个 slice **共享同一个底层数组**，所以做 `s2[i] = new(T)` 会把底层数组的第 i 个元素改掉，**老的也会受影响**。
- 如果你是用 `copy`/`append` 复制出**新的底层数组**，那 `s2[i] = ...` **不会影响** `s1[i]`。

**改指针指向的对象内容（解引用修改）**：

1. 只要 `s1[i]` 和 `s2[i]` 指向的是**同一个对象**，无论 slice 底层数组是否分开，`*s2[i] = ...` 这类修改都会让另一边看到变化。

 

只做 header 复制：共享底层数组 → 改元素会互相影响

**把这 3 个字段按值复制一份**（这就叫 header 复制）。

**底层数组没复制**，因此 `s1` 和 `s2` 的 `ptr` 指向同一块底层数组 → **共享元素槽位**。

```go
type N struct{ v int }
s1 := []*N{{v: 1}, {v: 2}}
s2 := s1              // 只复制 header，共享底层数组

s2[0] = &N{v: 100}    // 改的是“第0个元素存的指针值”
fmt.Println(s1[0].v)  // 100（因为同一底层数组的第0格被改了）

s1(header) ─┐
s2(header) ─┴──> [ slot0 | slot1 | slot2 ]   （同一个底层数组）
```

复制出新数组：不共享底层数组 → 改元素不影响

```go
s1 := []*N{{v: 1}, {v: 2}}

s2 := make([]*N, len(s1))
copy(s2, s1)          // 复制元素到新底层数组

s2[0] = &N{v: 100}    // 改的是 slice 的元素（指针值）
fmt.Println(s1[0].v)  // 1（s1 的第0格没被改）
```

即使底层数组分开，只要指向同一对象，改对象内容仍会互相影响

```go
s1 := []*N{{v: 1}, {v: 2}}

s2 := make([]*N, len(s1))
copy(s2, s1)          // 新数组，但元素里的指针仍指向同一批对象

s2[0].v = 999         // 改的是 指针指向的对象内容
fmt.Println(s1[0].v)  // 999（因为对象是同一个）
```

`copy(dst, src)` 做的是：**把 src 的元素逐个赋值到 dst 的底层数组里**。

关键点：**copy 不会替你分配 dst 的底层数组**。

dst 的底层数组来自你怎么创建 dst，例如：

```go
s2 := make([]*N, len(s1)) // 这一步才分配了“新的底层数组”
copy(s2, s1)              // 只是把元素(指针值)复制进去

s1 -> [ pA | pB ]
s2 -> [ pA | pB ]   （两个底层数组不同，但里面的指针值一开始相同）
```

**底层数组分开了” ≠ “指针指向的对象分开了**

`copy(s2, s1)` 之后，确实是：

- `1` 的底层数组 ≠ `s2` 的底层数组（槽位不共享）

- 但 `s1[i]` 和 `s2[i]` 里存的**指针值**可能相同（都指向同一块对象内存）

```
s1 -> [ pA | pB ]      （数组1：槽位）
          |     |
          v     v
        objA   objB    （对象）

s2 -> [ pA | pB ]      （数组2：槽位）
          |     |
          v     v
        objA   objB    （还是同一批对象）
```

所以：**数组分开了，但两边的指针仍指向同一个 objA/objB。**

槽位像“通讯录里的号码格子”，对象像“这个号码对应的人”。改号码≠改人。



如果想要互不影响，必须使用深拷贝

不仅要新数组，还要**新对象**：

```go
深拷贝：
s2 := make([]*N, len(s1))
for i, p := range s1 {
    if p != nil {
        x := *p      // 拷贝对象内容
        s2[i] = &x   // 指向新对象
    }
}
```



#### Map

map 是引用类型（底层是指针结构）

Go 的 Map 是一个 **Hash Table (哈希表)**，采用 **拉链法** (Buckets) 解决冲突。

- **hmap:** Map 的头部结构，存储了 count (元素个数)、B (桶的对数，决定桶的数量 $2^B$)等
- **bmap (Bucket):** 每一个桶最多存 **8 个键值对**。如果冲突超过 8 个，会通过 `overflow` 指针链接下一个溢出桶。

**map 不是线程安全**：并发读写会 panic

读一个不存在的 key：返回零值；

**用 `v, ok := m[k]` 判断是否存在**

```go
m := map[string]int{"a": 1}
v := m["b"]            // 0
v, ok := m["b"]        // 0, false
```

map初始化

```go
var m1 map[string]int  // nil map，不能写入
m2 := make(map[string]int) // 可以写入
```



1.**Map的底层实现原理是什么？**

Go 的 `map` 底层是 **哈希表（hash table）**。

运行时用一个 `hmap` 结构维护元信息，核心数据按 **桶（bucket）数组**存放：桶的数量始终是 `2^B`。

每个 bucket 里有固定数量（经典实现是 **8 个**）的 key/value 槽位，并带有 `tophash`（hash 高位摘要）用于快速过滤；

哈希冲突时通过 **overflow bucket** 链起来。

当装载因子过高或 overflow 过多时触发 **渐进式扩容（rehash/grow）**：

新旧桶数组并存，访问时逐步把旧桶搬迁到新桶，因此扩容不会一次性卡死。`map` 平均查找/插入是 O(1)。并发写（或读写并发）不安全。



2.**Go语言Map的遍历是有序的还是无序的？**

Go语言里Map的遍历是**完全随机**的，并没有固定的顺序。

map每次遍历,都会从一个随机值序号的桶,在每个桶中，再从按照之前选定随机槽位开始遍历,所以是无序的。



3.**Go语言Map的遍历为什么要设计成无序的？**

map 在扩容后，会发生 key 的搬迁，原来落在同一个 bucket 中的 key，搬迁后，有些 key 就要远走高飞了（bucket 序号加上了 2^B）。

而遍历的过程，就是按顺序遍历 bucket，同时按顺序遍历 bucket 中的 key。

搬迁后，key 的位置发生了重大的变化，有些 key 飞上高枝，有些 key 则原地不动。

这样，遍历 map 的结果就不可能按原来的顺序了。



4.**Map如何实现顺序读取？**

如果业务上确实需要有序遍历，最规范的做法就是将**Map的键（Key）取出来放入一个切片（Slice）中，用`sort`包对切片进行排序，然后根据这个有序的切片去遍历Map。**

```go
package main
import(
	"fmt"
	"sort"
)
func main(){
	keylist:=make([]int,0)
	m := map[int]int{
      3: 200,
      4: 200,
      1: 100,
      8: 800,
      5: 500,
      2: 200,
   }
    for key:=range m{
        keylist=append(keylist,key)
    }
    sort.Ints(keylist)
    for _,key:=range keylist{
        fmt.println(key,m[key])
    }
}
```



5.**Map是并发安全吗？**

不是并发安全

如果在多个 Goroutine 中同时对一个 `map` 进行读写（或者并发写），程序会直接崩溃，抛出无法 recover 的致命错误：

```go
fatal error: concurrent map writes
```

Go 官方团队（Rob Pike 等人）在设计时的考量是：

1. **性能优先：** 大多数情况下，Map 只在一个 Goroutine 内使用。如果为了支持并发而在底层强制加锁，会给那些不需要并发的场景带来巨大的性能损耗。
2. **并发检测：** Go 1.6 之后，Map 内部增加了**位标识**检测。在读写操作开始时，会检查这个标识。如果发现有其他协程正在写入，就会直接 Panic。

**在并发场景下该怎么用 Map？**

方案一：自带锁 (`sync.Mutex` 或 `sync.RWMutex`) —— **最通用**

这是最常见、最推荐的标准做法。自己封装一个结构体，把 Map 和锁绑在一起。

**优点：** 简单直观，适用所有场景。

**缺点：** 锁的粒度较粗。如果并发非常高，锁竞争（Lock Contention）会成为瓶颈。

方案二：Go 1.9 引入的 `sync.Map` —— **特定场景神器**

Go 官方提供了一个并发安全的 Map，但**它不是为了替代普通 Map + Mutex 而生的**。

**它的核心机制是“空间换时间”：** 内部维护了两个 Map：

1. **Read Map (只读表):** 原子操作 (`atomic.Value`)，不需要加锁，查找极快。
2. **Dirty Map (脏表):** 包含所有数据，写入需要加互斥锁。

**工作流程：**

- **读：** 先查 Read Map（无锁）。如果没找到，再去查 Dirty Map（加锁）。
- **写：** 直接写 Dirty Map。
- **数据同步：** 当 Dirty Map 被读取的次数足够多（Miss 次数达到阈值），系统会将 Dirty Map 提升为 Read Map。

**什么时候用 `sync.Map`？** 官方文档明确指出，只有两种情况比 `Mutex` 快：

1. **读多写少：** 也就是 Key 写入一次，读取很多次（如缓存）。
2. **多协程写不同 Key：** 多个协程分别写入不相交的 Key 集合。

**缺点：** 如果是**频繁的写入/更新**，`sync.Map` 性能反而不如方案一，因为需要不断地在 Read 和 Dirty 之间倒腾数据。

方案三：分片锁 (Concurrent Map) —— **高性能黑科技**

如果在极高并发下（比如百万级 QPS），一把大锁（方案一）会卡死，`sync.Map`（方案二）写入太慢，怎么办？

**思路：** **将一个大 Map 拆分成 N 个小 Map（分片/Sharding）**。 比如定义 32 个 Map，根据 Key 的 Hash 值决定数据落到哪个 Map 上。

- **优点：** 极大降低了锁的粒度，并发性能极高（类似 Java 的 `ConcurrentHashMap` 早期实现）。
- **开源库：** 很多知名项目使用 `orcaman/concurrent-map`。

> 另外，在开发和测试阶段，使用 `go run -race` 来自动检测潜在的并发 Map 访问问题。"



6**.Map的Key一定要是可比较的吗？为什么？**

**Map的Key必须要可比较。**

首先，**Map会对我们提供的Key进行哈希运算，得到一个哈希值。**

**这个哈希值决定了这个键值对大概存储在哪个位置（也就是哪个"桶"里）。**

然而，**不同的Key可能会产生相同的哈希值，这就是"哈希冲突"。**

**当多个Key被定位到同一个"桶"里时，Map就没法只靠哈希值来区分它们了。**

此时，它必须**在桶内进行逐个遍历，用我们传入的Key和桶里已有的每一个Key进行相等（==）比较**。

**这样才能确保我们操作的是正确的键值对。**



7.Map可以边遍历边删除吗

不行，Map不是并发安全的数据结构。

如果多个线程边遍历，边删除，同时读写一个 map 是未定义的行为，如果被检测到，会直接 panic。



如果是发生在多个协程同时读写同一个 map 的情况下。 

如果在同一个协程内边遍历边删除，并不会检测到同时读写，理论上是可以这样做的。

但是，遍历的结果就可能不会是相同的了，有可能结果遍历结果集中包含了删除的 key，也有可能不包含，这取决于删除 key 的时间：是在遍历到 key 所在的 bucket 时刻前或者后。这种情况 下，可以通过加读写锁sync.RWMutex来保证





### 指针、方法接收者（值接收者 vs 指针接收者）



### 错误处理：`error`、自定义 error、`panic/recover` 的边界

在 Java 或 Python 中，你可能习惯了 `try-catch` 的异常流控制。但在 Go 的世界里，你要记住第一条军规：**Error is a Value（错误即值）**。它不是一个“异常”，它只是一个普通的返回值，就像 `int` 或 `string` 一样。

作为后端开发，如何优雅地处理错误，直接决定了你的服务是“健壮如牛”还是“一碰就倒”。

我们分三个层次来讲：**基础与标准库**、**自定义与错误链**、**核武器 Panic/Recover**。

#### 第一层：Error 的本质与标准姿势

##### 1. 什么是 `error`？

它不是关键字，而是一个内置的**接口 (Interface)**。

```go
// 源码定义
type error interface {
    Error() string
}
```

这意味着：**任何实现了 `Error() string` 方法的类型，都可以被当作 `error` 使用。**

##### 2. 创建错误的两种方式

- **静态错误 (Sentinel Errors):** 预定义的、可重用的错误。

	```go
	import "errors"
	var ErrNotFound = errors.New("user not found") // 通常以 Err 开头
	```

- **动态错误:** 带有上下文信息的错误。

	```go
	import "fmt"
	return fmt.Errorf("database query failed: %s", sqlQuery)
	```

#### 第二层：Go 1.13+ 的错误链 (Wrapping) —— 面试加分项

在微服务后端，仅仅返回 `errors.New("db error")` 是不够的。

我们需要知道：**哪一层调用的？参数是什么？根因是什么？**

这就引入了 **Error Wrapping (错误包装)** 的概念。

想象一下，错误就像一个洋葱，我们一层层把上下文包上去。

##### 1. 如何包装？ (`%w`)

使用 `fmt.Errorf` 配合 `%w` (wrap) 动词。

```go
func DaoLayer() error {
    return errors.New("connection timeout") // 根因
}

func ServiceLayer() error {
    err := DaoLayer()
    if err != nil {
        // 🌟 重点：使用 %w 把原始错误包起来，加上这层的上下文
        return fmt.Errorf("service query failed: %w", err)
    }
    return nil
}
```

##### 2. 如何判断？ (`Is` 和 `As`)

以前我们用 `if err == ErrNotFound`，现在有了包装，`==` 就不灵了（因为包了一层）。

**`errors.Is` (判断是不是某种特定的错误值):** 会自动剥开洋葱皮去比对。

```go
if errors.Is(err, ErrConnectionTimeout) {
    // 处理超时
}
```

**`errors.As` (判断是不是某种特定的错误类型):** 相当于类型断言 (`Type Assertion`)。

```go
var mysqlErr *MySqlError
if errors.As(err, &mysqlErr) {
    // 现在可以访问 mysqlErr.Code 了
    fmt.Println(mysqlErr.ErrorCode)
}
```

#### 第三层：自定义 Error (工程化实践)

在真实的后端项目中，我们通常需要携带 **错误码 (Error Code)** 给前端，或者携带 **HTTP 状态码**。

```go
type AppError struct {
    Code    int    // 业务错误码，如 10001
    Message string // 提示信息
    Err     error  // 原始错误（用于记录日志，不返回给前端）
}

// 必须实现 Error() 接口
func (e *AppError) Error() string {
    return fmt.Sprintf("code=%d, msg=%s", e.Code, e.Message)
}

// 让我们支持 Unwrap
func (e *AppError) Unwrap() error {
    return e.Err
}
```

#### 第四层：Panic 与 Recover —— 既然有 Error，为什么还要 Panic？

##### 1. 什么时候用 Panic？

Go 的哲学是：

**普通错误用 `error`，不可恢复的致命错误用 `panic`。**

- **✅ 该 Panic 的场景：**
	- 程序启动时，关键配置文件缺失（没配置怎么跑？直接挂掉提醒运维）。
	- 数组越界（这是代码逻辑 bug，必须修复，不能吞掉）。
	- 空指针引用。
- **❌ 不该 Panic 的场景：**
	- 数据库连接失败（应该重试或返回错误）。
	- 用户输入不合法（返回 400 错误）。

##### 2. Recover 的正确姿势

`recover` 只能在 `defer` 中生效。

它的作用是：**把即将崩溃的程序“拉”回来，通常转换为一个 `error` 返回。**

```go
func SafeHandler() {
    defer func() {
        if r := recover(); r != nil {
            // 🌟 捕获到了 Panic！
            fmt.Println("Recovered from panic:", r)
            // 这里通常会记录日志，并返回 500 错误给前端
        }
    }()
    
    // 可能发生 Panic 的逻辑
    var slice []int
    fmt.Println(slice[0]) // Panic!
}
```

**Panic/Recover 的边界（面试杀手题）**

**面试题：** *“我在主 Goroutine 里写了 `defer recover`，能捕获到**子 Goroutine 里的 Panic** 吗？”*

**答案：不能！绝对不能！**

这是 Go 后端开发中最危险的坑。**Panic 只会触发当前 Goroutine 的栈展开。** 如果你开了一个协程去处理任务，那个协程 Panic 了且没有自己的 recover，**整个进程（整个 App）都会直接崩溃**。

```go
func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Main recover") // 👈 这里永远不会执行
        }
    }()

    go func() {
        panic("I am killing the whole app!") // 👈 子协程 Panic
    }()

    time.Sleep(1 * time.Second)
}
```

**解决方案：** 在每一个你启动的 `go func()` 开头，都必须加上 `defer recover`。通常我们会封装一个 `GoSafe()` 函数来做这件事。



### 常用标准库：`context`、`net/http`、`time`、`encoding/json`









## 并发编程

### goroutine / channel 基本使用

### channel 缓冲 vs 无缓冲、关闭 close 的语义

### `select`、超时控制

### 同步原语：`sync.Mutex / RWMutex / WaitGroup / Once / Cond`/Map

### 原子操作 `sync/atomic`

### 常见并发模式：worker pool、fan-in/fan-out、pipeline

### **context 贯穿**：取消、超时、传递 trace id（很后端）

## Web 服务开发

### 以 `net/http` 为根，再学框架（Gin / Echo）会更稳。

- HTTP 基础：请求/响应、header、cookie、status code
- RESTful API 设计（路由、参数、分页、错误码规范）
- 中间件（logging、auth、recover、cors、timeout）
- 参数校验、统一返回格式
- graceful shutdown（优雅停机）
- 结构化日志（zap / logrus，至少会一种）
- 配置管理（viper 或环境变量）

## 数据库与持久化

## 工程化与可维护性

- 项目目录结构（分层：handler/service/repo）
- 单元测试：`testing`、表驱动测试、mock 思路
- 性能分析：`pprof`（至少知道怎么抓 CPU/内存）
- CI/CD 基本概念
- Docker 基础（会写 Dockerfile、会构建运行）
- Git 基本操作、分支模型

## 分布式与中间件

RPC（gRPC 基本概念、IDL、拦截器）

消息队列（Kafka / RabbitMQ）：削峰填谷、重试、幂等、顺序

认证授权：JWT / session

限流、熔断、降级（知道场景 + 常见方案）

可观测性：metrics、trace、log（三件套）