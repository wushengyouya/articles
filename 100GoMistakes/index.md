---
author: 吴生有崖
title: 100GoMistakes
date: 2026-01-05
description: 100 Go Mistakes And How To Avoid Them笔记
image: 
categories:
  - 编程
tags:
  - go
---
## 100 Go Mistakes And How To Avoid Them

1、避免使用遮挡变量
```go
var i
func get(){
    i := 1
}
```

2、通过避免嵌套层级并将主路径保持在左侧

3、init函数不能处理error，当有多个init函数先加载依赖项的init(按照文件名字母表的顺序)

4、抽象是被发现的，而不是创建的。

5、接口尽量在使用端，而不是在生产端

6、函数不应该返回抽象而是具体的实现，而参数尽可能的使用接口作为参数

7、尽量不使用`any`类型，因为它提供有意思的信息，尽在需要进行序列化的使用`any`

8、当有多个参数需要配置时，可以采用功能选项模式。(function option)

9、创建包名时应该可以直观的明白这个包时做什么的，避免使用泛化包名如util、base、common

10、对于可以在外部访问的变量与函数都需要注释
- 变量和常量: 注释用途和内容
- 函数和方法：注释函数用途而非实现方式

11、使用静态代码分析工具检测代码如[vet](https://golang.org/cmd/vet/)、[golangci-lint](https://golangci-lint.run/docs/linters/)

12、在go中`0o`代表8进制,`0B | 0b`代表二进制,`0x`代表十六进制,`i`后缀代表虚数

13、go中整数上溢和下溢都是隐式的，可以使用`math.MaxInt`,`math.MinInt`,`math.MaxUint`构建函数检测

14、浮点数在加减运算时应将数量级相近的操作分组以提高准确性，乘除运算应在加减运算前进行以提高准确性

## 切片
- 切片的长度是当前可以获取的元素数，容量是可以存储的元素数。当元素存储满时底层会创建一个新的数组，容量在元素在1024以内成倍扩容，超过后每次扩容1/4
- 当创建切片时如果已知长度，make时指定长度，减少内存分配提升性能。同样适用于创建map的时候
- 复制一个切片到另一个切片使用`copy`函数，复制的元素数取决于两个切片中最小的长度
- `str[1:5]`使用切片表达式缩减切片，底层都是指向同一个数组，如果缩减一个大切片时可能会发生内存泄漏。可以使用`copy`或   
完整的切片表达式`str[1:5:2]`缩减大切片必须使用`copy`
- 在处理切片时需要记住，如果元素是指针或者是包含指针资源的结构体，这些元素将不会被gc回收，使用下面方法避免内存泄漏。  
  清除方法: 1.手动置为nil 2.使用`copy`而不是reslice
```go
// 示例1：指针切片
// 创建包含指针的切片
items := make([]*Item, 0, 100)

// 添加一些指针元素
items = append(items, &Item{data: "data1"})
items = append(items, &Item{data: "data2"})

// 假设我们"删除"第一个元素
// 手动清除方法 items[0] = nil
// 使用copy
copy(item[0:],item[1:])
items = items[1:]  // 现在 items 只有 data2
// 问题：底层数组仍然引用着 data1 的指针！
// data1 指向的内存不会被GC回收

///////////////////////////////////////////////////////
// 示例2：包含指针字段的结构体
// 问题：底层数组仍然引用着 data1 的指针！
// data1 指向的内存不会被GC回收
type User struct {
    Name string
    Data *Data  // 指针字段！
}

users := make([]User, 0, 10)
users = append(users, User{Name: "Alice", Data: &Data{value: 100}})

// 删除元素后，Data指针仍然被底层数组引用
users = users[1:]
```
- 使用`append`后，如果结果切片的长度小于容量，会改变原始切片
```go
func main(){
  s := []int{1, 2, 3}
  f(s[:2])
  fmt.Println(s) // [1,2,10]
  
}

func f(s []int){
  _ = append(s, 10)
}

/*
// append操作分析：
1. 检查容量：len=2, cap=3，容量足够
2. 不创建新数组，直接使用原底层数组
3. 在原底层数组索引2位置写入10
4. 返回新切片：len=3, cap=3

内存布局变化:
索引:  0   1   2
前:   [1] [2] [3]
后:   [1] [2] [10]  // 索引2的值被覆盖了！

// 注意：_ = append(s, 10)
// 虽然我们忽略了返回值，但底层数组已经被修改了！
*/
```
- 记住多个切片可能共享同一个底层数组
- 检查切片是否包含元素使用`len`函数，nil与empty切片都返回0.同样适用于map

16、map只会增长不会进行收缩，如果要避免内存泄漏，可以使用指针存储或者创建一个新map

17、使用`==`或`!=`可以对boolean,numeral,string,pointer,channel,struct进行比较，对于是maps和slice可以使用`reflect.DeepEqual`进行比较

18、`for...range`中所有值皆为复制的，如果想要修改值使用索引访问，或者是循环的指针

19、`range` 后面的值当执行时会复制一次,无论其是什么类型都是复制的值

29、`for...range`中所有数据值都会被分配到具有唯一地址的变量中。因此，如果每次遍历时都存储指向该变量的指针，就会陷入“同一指针指向同一元素”的循环陷阱
——即始终指向最新元素。解决方法有两种：要么在循环作用域内强制创建局部变量，要么通过索引创建指向切片元素的指针。

30、map进行迭代和插入都是无序的，无法保证顺序。
- 假设对一个map插入abcde,迭代出abced cedba都是可能的
  
31、不要在循环中使用`defer`，它会将语句在函数返回之后再执行。如果一定要在循环中使用`defer`可以在循环中调用一个函数比如闭包

32、go中`string`底层引用的是一个不能修改`byte`切片,字符串使用`len`返回的是字节数而不实字符数，如果想要返回字符串需要将`string`转为`rune`
统计字符数可以使用`utf8.RuneCountInString`
```go
s := "hello"
fmt.Println(len(s)) // 5

s := "汉"
fmt.Println(len(s)) // 3
```
33、go的源码都采用utf-8进行编码

34、若要遍历字符串中的`rune`，可以直接使用字符串的范围循环。但需注意，这里的索引并非`rune`本身的索引，而是底层字节序列的起始位置。由于`rune`可能由多个字节组成，若要访问`rune`本身，应使用范围循环的值变量而非字符串中的索引。此外，若要获取字符串的第i个符文，在大多数情况下需要将字符串转换为`rune`切片。
```go
s:="adbcd"
r:= []rune(s)

// 获取字符串长度
utf8.RuneCountInString(s)

```

34、`TrimRight`或`TrimLeft`函数会逐个回溯遍历字符串中的每个字符。若某个字符属于给定集合，则被移除；否则终止遍历并返回剩余字符串。因此本示例返回123。
```go
fmt.Println(strings.TrimRight("123oxo", "xo")) // 123
fmt.Println(strings.TrimLeft("oxo123", "ox")) // 123
fmt.Println(strings.TrimPrefix("oxo123", "ox")) /// o123
```

35、go中字符串是不可变的如果需要进行多个字符串拼接使用`strings.Builder`，若已知未来字符串的字节数，应使用Grow方法预先分配内部字节切片
```go
func concat(values []string) string {
  total := 0
  for i := 0; i < len(values); i++ {
    total += len(values[i]) // 统计每个字符串的字节数
  }
  sb := strings.Builder{} // 底层是持有一个byte切片,WriteString执行后都会对这个切片进行追加操作
  // Grow 给builder持有的切片指定多少字节数，减少字节切片扩容操作，提升性能
  sb.Grow(total) // 根据统计的字节数给builder开辟空间，减少内存分配
  for _, value := range values {
    _, _ = sb.WriteString(value)
  }
  return sb.String()
}
```

36、大多数输入输出操作都是用[]byte而不是字符串来完成的。当我们纠结该用字符串还是[]byte时，不妨回想一下：使用[]byte其实并不一定不方便。事实上，字符串包里所有导出的函数在bytes包里都有对应版本，比如Split、Count、Contains、Index等等。因此，无论是否进行I/O操作，我们都应该先考虑是否可以用bytes替代字符串来实现整个工作流程，从而避免额外转换带来的麻烦。

37、字符串需要记住 1.`len`返回的是字节数而不实字符串 2.对字符串进行切割子串操作可能会产生内存泄漏(`log[:5]`),从go 1.18开始创建一个字符串的子串可以使用`strings.Clone`

38、方法接收器
- 必须为指针   
  
  1.若方法需要对接收者进行类型转换。当接收者为切片且方法需要追加元素时，该规则同样适用。   
  2.若方法接收方包含不可复制的字段（例如sync包的类型部分），我们将在错误#74《复制同步类型》中详细讨论该问题。
```go
type slice []int
func (s *slice) add(element int) {
*s = append(*s, element)
}
```   
- 应该为指针

若接收方为大型对象，使用指针可提升调用效率，因其能避免进行大规模数据复制。当难以界定‘大型’的具体范围时，基准测试可作为解决方案；由于其受多重因素影响，几乎无法给出确切的尺寸标准。

- 必须为值类型   
  
  1.如果必须强制接收方不可变性。    
  2.如果接收方是映射、函数或通道。否则将发生编译错误。

- 应该为值类型

  1.如果接收者是一个不需要修改的切片。    
  2.如果接收者是一个小型数组或结构体，其本身是值类型且没有可变字段，例如时间。   
  3.如果接收者是基本类型，如int、float64或string。

39、函数使用命令返回值时，默认会初始化的为该类型的初始值

40、在大多数情况下，将文件名作为函数参数来读取文件内容应被视为一种代码异味（特定函数如os.Open除外）。可以使用io.Reader作为参数
```go
func countEmptyLines(reader io.Reader) (int, error) {
scanner := bufio.NewScanner(reader)
  for scanner.Scan() {
  // ...
  }
}
```

41.使用`defer`调用函数时，参数为立刻进行赋值。简而言之，调用函数或方法的defer操作时，其参数会立即执行。若需在后续修改传入的参数，可使用指针或闭包实现。
```go
s := "abc"
defer f(s) // s = abc
defer f(){
  fmt.Println(s) // s = ddd 闭包调用执行时才赋值
}()
s = "ddd"

func main() {
  i := 0
  j := 0
  defer func(i int) {
  fmt.Println(i, j)
  }(i)
  i++
  j++
  // i = 0 
  // j = 1
}
```

42.在处理错误时，我们可以选择进行错误包装。包装是指为错误添加额外上下文信息和/或将错误标记为特定类型。如果需要对错误进行标记，则应创建自定义错误类型。然而，若只需添加额外上下文，就应使用带有%w指令的fmt.Errorf，因为这无需创建新的错误类型。但需注意，错误包装可能产生耦合性，因为它会使调用方能够访问原始错误。若要避免这种情况，就不应使用错误包装，而应采用错误转换，例如使用带有%v指令的fmt.Errorf。
```go
// 错误包装
if err != nil {
  return fmt.Errorf("bar failed: %w", err)
}
```

43.如果我们依赖Go 1.13的错误包装机制，就必须使用`errors.As`来检查某个错误是否属于特定类型。这样一来，无论错误是由被调用函数直接返回，还是被包装在另一个错误中，`errors.As`都能递归解包主错误，并判断其中是否存在符合特定类型的错误。

44.若在应用程序中使用错误包裹功能（通过%w指令和fmt错误函数），则应使用`errors.Is`而非==来检查错误是否与特定值匹配。因此，即使哨兵错误被包裹，`errors.Is`仍能递归解包，并将链中的每个错误与指定值进行比对。

45.错误处理应当只执行一次。正如我们所见，记录错误本质上就是处理错误。因此，我们应当选择记录错误或返回错误。这样做既能简化代码，又能更深入地理解错误情况。使用错误包装是最便捷的方法，它既能传递源错误，又能为错误添加上下文信息。

46.当想要忽略一个err时，使用`_`占位更能传达出明显的有效信息,表示忽略处理这个错误

47.并发是指同时处理多项任务。并行是指同时执行多项任务。

48.在开发并发应用程序时，必须明确区分数据竞争与竞争条件。当多个goroutine同时访问同一内存地址且至少有一个在写入时，就会发生数据竞争。这种现象会导致程序出现异常行为。但需要注意的是，无数据竞争的应用程序并不意味着结果绝对确定。即使程序没有数据竞争，仍可能因goroutine执行速度、消息发布到通道的时效性、数据库调用耗时等不可控因素产生波动——这正是竞争条件的表现。准确理解这两个概念，是掌握并发应用设计精髓的关键所在。

49.如果传输context不确定时使用context.TODO()，context必须要保证一直存在直到函数返回

50.创建一个go程必须清楚的知道它什么时候执行关闭,必须一直占用资源

51.在使用带多个通道的select语句时，必须注意：当多个选项同时就绪时，源码中的第一个case不会自动胜出。Go会随机挑选可执行的case，因此无法保证具体选中哪个选项。针对单个生产者协程的情况，可通过无缓冲通道或单一通道解决此问题；而对于多个生产者协程的场景，则可通过嵌套select配合default子句来实现优先级处理。

52.通道可以携带数据或不携带数据。若希望按照Go语言惯例设计符合习惯的API，需注意：不携带数据的通道应声明为`chan struct{}`类型。这种设计能向接收方明确传达：消息本身不包含任何有意义的内容，其价值仅在于传递"已接收到信号"这一事实。在Go语言中，此类通道被称为**通知通道**。

53.总结来说，等待一个nil通道或向nil通道发送数据会导致阻塞，而这一特性并非无用。正如我们在合并两个通道的示例中所见，可以利用nil通道来实现一种精巧的状态机，从而动态地从select语句中移除其中一个分支。让我们记住这个思路：nil通道在特定场景下非常有用，应成为Go开发者在处理并发代码时的工具之一。
```go
// 示例：使用nil通道动态管理select分支
for {
    select {
    case v, ok := <-ch1:
        if !ok {
            ch1 = nil // 将通道置为nil，使此case不再被选中
            continue
        }
        process(v)
    case v, ok := <-ch2:
        if !ok {
            ch2 = nil // 将通道置为nil，使此case不再被选中
            continue
        }
        process(v)
    }

    if ch1 == nil && ch2 == nil {
        break // 两个通道都已关闭，退出循环
    }
}
```
54.在并发中使用字符串格式化可能会导致`数据竞争`与`死锁`
```go
func potentialDeadlock() {
    var mu sync.Mutex
    
    mu.Lock()
    // 危险：如果format内部触发了需要同一锁的操作
    s := fmt.Sprintf("Locked: %v", someFunction()) 
    mu.Unlock()
}

func someFunction() string {
    // 如果这里也需要获取同一个锁，就会死锁
    mu.Lock() // 第二次尝试加锁 - 死锁！
    defer mu.Unlock()
    return "data"
}

//// 数据竞争
var sharedData string

func goroutine1() {
    // 不安全：同时修改共享数据
    sharedData = fmt.Sprintf("Value: %d", time.Now().Unix())
}

func goroutine2() {
    // 同时读取或修改 sharedData
    fmt.Println(sharedData)
}
```

55.在并发环境中处理切片时，我们必须记住，对切片使用 `append` 操作并不总是无竞态的。根据切片的状态以及它是否已满，行为会发生变化。如果切片已满，`append` 操作是无竞态的。否则，多个 goroutine 可能会竞争更新同一个数组索引，从而导致数据竞态。
```go
s := make([]int, 1) // 无数据竟态,当一个go程插入后，切片已满底层会重新创建一个数组扩容
s := make([]int, 0, 1) // 存在数据竟态,两个go程都访问索引为 1 的位置
go func() {
  s1 := append(s, 1)
  fmt.Println(s1)
}()
go func() {
  s2 := append(s, 1)
  fmt.Println(s2)
}()
```

56.当需要触发多个goroutine并处理错误及上下文传递时，可考虑errgroup是否为一种解决方案。
```go
func handler(ctx context.Context, circles []Circle) ([]Result, error) {
  results := make([]Result, len(circles))
  g, ctx := errgroup.WithContext(ctx)
  for i, circle := range circles {
    i := i
    circle := circle
    g.Go(func() error {
      result, err := foo(ctx, circle)
        if err != nil {
          return err
        }
        results[i] = result
        return nil
    })
  }
  if err := g.Wait(); err != nil { // g.Wait() 等待所有go程执行完成
    return nil, err
  }
  return results, nil
}
```

57.使用`sync`包时，任何时候不要使用复制值。当多个goroutine需要访问同一同步元素时，必须确保它们都依赖于同一实例。该规则适用于同步包中定义的所有类型。使用指针是解决此问题的方法：我们可以使用指向同步元素的指针，或指向包含同步元素的结构体的指针。
```go
// 都不能被复制
sync.Cond
sync.Map
sync.Mutex
sync.RWMutex
sync.Once
sync.Pool
sync.WaitGroup
```
在以下情况下，我们可能会遇到意外复制sync字段的问题：
- 调用具有值接收器的方法（如前所述）
- 调用接收sync类型参数的函数
- 调用接收包含sync字段的结构体参数的函数

58.通常使用time.After方法时需谨慎。需注意资源仅在计时器到期后才会释放。若在循环、Kafka消费者函数或HTTP处理程序中重复调用time.After，可能导致内存消耗激增。此时应优先选用time.NewTimer。
```go
func consumer(ch <-chan Event) {
  timerDuration := 1 * time.Hour
  timer := time.NewTimer(timerDuration)
  for {
    timer.Reset(timerDuration)
    select {
      case event := <-ch:
      handle(event)
      case <-timer.C:
      log.Println("warning: no messages received")
    }
  }
}
```

59.我们应当谨慎处理嵌入字段。虽然推广嵌入字段类型的字段和方法有时会带来便利，但也可能引发细微错误，因为这可能导致父结构体在缺乏明确信号的情况下实现接口。
```go
// json格式化 会有问题 ID字段会被省略 
// 执行json.Marshal()后  "2021-05-18T21:15:08.381652+02:00"
type Event struct {
  ID int
  time.Time // 这样会组合会实现time的所有方法,  time.Time实现了json.Marshaler当调用json.Marshal()会调用它已实现的方法
}
type Event struct {
  ID int
  Time time.Time // 使用
}
// 
```

60.反序列化json字符串可以使用`map[string]any`,需要注意的是使用任何数值类型（无论是否包含小数）时，系统都会将其转换为float64类型。

## 数据库
- `sql.Open`方法并不强制建立连接，首个连接可采用延迟建立机制。若需验证配置正确性并确认数据库连接状态，应在调用`sql.Open`后执行`Ping`或`PingContext`方法。  
- 同样重要的是要记住，创建连接池会涉及四个可配置参数，我们可能需要对它们进行自定义。这些参数分别对应 *sql.DB 的四个导出方法：  
  1.SetMaxOpenConns：对于生产级应用至关重要。由于默认值是`无限制`的，我们应通过设置该参数来确保连接数符合底层数据库的实际承载能力。  
  2.SetMaxIdleConns：如果应用程序产生大量并发请求，应提高SetMaxIdleConns的默认值（默认为2），否则应用可能会频繁重新建立连接。  
  3.SetConnMaxIdleTime：若应用程序可能面临突发请求，设置SetConnMaxIdleTime十分重要(默认无限制)。当应用恢复平稳状态时，我们需要确保已创建的连接最终被释放。  
  4.SetConnMaxLifetime：在连接负载均衡数据库服务器等场景下，设置SetConnMaxLifetime会很有帮助(默认无限制)。这能确保应用程序不会过长时间占用单个连接。
- 预编译语句是许多SQL数据库为执行重复SQL语句而实现的功能。在内部，该SQL语句经过预编译并与提供的数据分离。
  ```go
  // 如果某个sql语句需要重复执行可以使用预编译语句
  // 优势是更有效率，更安全
  stmt, err := db.Prepare("SELECT * FROM ORDER WHERE ID = ?")
  ```
- 处理查询中的空值。字段为指针，或使用SQL的NullXXX类型。`sql.NullString` `sql.NullBool`
- 处理迭代中的行错误使用`row.Err`
  ```go
  func get(ctx context.Context, db *sql.DB, id string) (string, int, error) {
    // ...
    for rows.Next() {
    // ...
    }
    if err := rows.Err(); err != nil {
      return "", 0, err
    }
    return department, age, nil
  }
  ```

62.所有实现`io.Closer`资源结构在使用完后都需要被关闭，不然可能导致内存泄漏或其他的问题。如`HTTP body` `sql.Rows` `os.File`
```go
type Closer interface {
  Close() error
}
```

63.在生产级应用中，必须避免使用默认的HTTP客户端和服务器。否则，由于缺乏超时机制，甚至存在恶意客户端利用服务器无超时限制的漏洞，可能导致请求无限期卡住。

```go
client := &http.Client{
  Timeout: 5 * time.Second, // 请求超时等待时间
  Transport: &http.Transport{
      DialContext: (&net.Dialer{
        Timeout: time.Second, // 连接超时等待时间
      }).DialContext,
      TLSHandshakeTimeout: time.Second, // tls握手超时等待时间
      ResponseHeaderTimeout: time.Second, // 服务器响应头等待时间
  },
}
s := &http.Server{
  Addr: ":8080",
  ReadHeaderTimeout: 500 * time.Millisecond, // 读取请求头等待超时
  ReadTimeout: 500 * time.Millisecond, // 读取请求超时
  Handler: http.TimeoutHandler(handler, time.Second, "foo"), // 一个封装函数，用于指定处理程序完成的最大时间
}
```

## 测试
- 给测试文件添加标识区分  
  1.添加标识 `//go:build integration`      
  2.使用短测试 `testing.Short()`  
  3.使用环境变量标记
```go
//go:build integration
package db
import (
"testing"
)
func TestInsert(t *testing.T) {
// ...
}
// $ go test --tags=integration -v .
```
//go:build !integration 使用了`!`  
若使用integration标签运行go test，则仅执行集成测试。    
若不使用该标签运行go test，则仅执行单元测试。  

- 一种测试分类方法涉及运行速度。我们需要区分短时运行测试与长时运行测试。举例来说，假设我们有一组单元测试，其中某个测试运行速度极慢。我们希望对这个慢速测试进行分类，避免每次运行（特别是当触发条件是文件保存后时）。短时运行模式使我们能够实现这种区分
```go
func TestLongRunning(t *testing.T) {
  if testing.Short() {
  t.Skip("skipping long-running test")
  }
  // ...
}
```

- 我们应当牢记：对于使用并发的应用程序，强烈建议（甚至必须）在运行时启用-race参数。该参数可激活数据竞争检测器，通过代码监控来捕捉潜在的数据竞争。
```shell
go test -race ./...
```

- 当使用`t. Parallel`标记测试时，该测试会与所有其他并行测试同时执行。但在执行流程中，Go会先逐一运行所有顺序测试，待顺序测试完成后，再执行并行测试。
```go
// 先执行C 再并行执行 AB
func TestA(t *testing.T) {
  t.Parallel()
  // ...
}
func TestB(t *testing.T) {
  t.Parallel()
  // ...
}
func TestC(t *testing.T) {
  // ...
}
```

- 表格驱动测试是一种高效编写精简测试的技巧。若多个单元测试具有相似结构，可采用表格驱动测试实现互化。该技术通过避免重复，简化了测试逻辑的修改，并便于新增用例。
```go
func TestFoo(t *testing.T) {
  t.Run("subtest 1", func(t *testing.T) {
    if false {
      t.Error()
    }
  })
  t.Run("subtest 2", func(t *testing.T) {
    if 2 != 2 {
      t.Error()
    }
  })
}
```

- httptest软件包（https://pkg.go.dev/net/http/httptest）为HTTP测试提供客户端和服务器端的工具
```go
// 模拟请求
req := httptest.NewRequest(http.MethodGet, "http://localhost",
strings.NewReader("foo"))

// 模拟服务器
srv := httptest.NewServer(
  http.HandlerFunc(
  func(w http.ResponseWriter, r *http.Request) {
    _, _ = w.Write([]byte(`{"duration": 314}`))
    },
  ),
)
defer srv.Close()
```

- iotest软件包（https://pkg.go.dev/testing/iotest）提供了测试读写器的实用工具。这个便捷的工具包常被Go开发者忽视。
```go
func TestLowerCaseReader(t *testing.T) {
  err := iotest.TestReader(
    &LowerCaseReader{reader: strings.NewReader("aBcDeFgHiJ")},
    []byte("acegi"),
  )
  if err != nil {
    t.Fatal(err)
  }
}
```

- 查看测试覆盖率
```shell
$ go test -coverprofile=coverage.out ./...
$ go tool cover -html=coverage.out
```

- 测试环境准备，在测试前准备资源，在测试后关闭资源
```go
// 测试前setup
func TestMySQLIntegration(t *testing.T) {
  // ...
  db := createConnection(t, "tcp(localhost:3306)/db")
  // ...
}

func createConnection(t *testing.T, dsn string) *sql.DB {
  db, err := sql.Open("mysql", dsn)
  if err != nil {
    t.FailNow()
  }
  t.Cleanup( // 测试完后关闭资源
    func() {
      _ = db.Close()
  })
  return db
}
```
- 该特定函数接受一个`*testing.M`参数，该参数通过暴露单一的Run方法来执行所有测试。
```go

func TestMain(m *testing.M) {
  setupMySQL()
  code := m.Run()
  teardownMySQL()
  os.Exit(code)
}
```

64.设计结构体时如何减少内存分配量？经验法则是对结构体进行重组，使其字段按类型大小降序排列。在本案例中，int64类型排在首位，随后是两个字节类型：
```go
// 由于结构体的大小必须是字长（8字节）的整数倍，因此其地址总长度为24字节而非17字节。编译时，Go编译器会添加填充数据以确保数据对齐
type Foo struct {
  i int64
  b1 byte // 在64位的系统编译器默认会补 7个byte
  b2 byte
}
```
设计结构图需注意数据对齐问题。将Go结构体的字段按大小降序排列可避免填充。防止填充意味着分配更紧凑的结构体，这可能带来诸如减少垃圾回收频率和提升空间局部性等优化效果。

65.若编译器无法确认变量在函数返回后未被引用，则该变量将被分配到堆内存中。
- 函数或方法返回指针，变量会逃逸到堆中

66.编译优化
- map优化
```go
// 这个版本要快些，编译器会避免将bytes转为string
func (c *cache) get(bytes []byte) (v int, contains bool) {
  v, contains = c.m[string(bytes)]
  return
}
func (c *cache) get(bytes []byte) (v int, contains bool) {
  key := string(bytes)
  v, contains = c.m[key]
  return
}
```
- sync.pool   
  当我们需要频繁创建大量同一类型的对象时，可以考虑使用`sync.Pool`。作为一组可复用的临时对象存储池，它能有效避免同类数据的重复内存分配，并且支持多个goroutine安全并发访问。
```go
// 需要创建多个byte切片的场景使用sync.pool优化
var pool = sync.Pool{
  New: func() any {
  return make([]byte, 1024)
  },
}
func write(w io.Writer) {
  buffer := pool.Get().([]byte)
  buffer = buffer[:0]// 清空buffer
  defer pool.Put(buffer)
  getResponse(buffer)
  _, _ = w.Write(buffer)
}
```
67.使用pprof检测
```go
package main
import (
  "fmt"
  "log"
  "net/http"
  _ "net/http/pprof"
)
func main() {
  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) { 
  fmt.Fprintf(w, "")
  })
  log.Fatal(http.ListenAndServe(":80", nil))
}
```
