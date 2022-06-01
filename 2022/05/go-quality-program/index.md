# Golang高质量编程与性能调优实战


## Golang高质量编程与性能调优实战 | 青训笔记

这是我参与「第三届青训营 -后端场」笔记创作活动的的第3篇笔记。 

本篇blog将对前面学习的Go by Example最后一部分进行归纳总结，主要内容有函数自定义排序、文件读写以及命令行相关参数等等。同时对如何进行高质量编程，Go程序优化手段和pprof性能分析工具的使用进行学习总结。完整代码见[Github](https://github.com/KevinChens/youth-camp/tree/main/lesson3)。  

### Go by Example
1.  `sort`包实现了内建及用户自定义数据类型的排序功能。它是原地排序，不返回一个新切片。  
```go
strs := []string{"c", "a", "b"}
sort.Strings(strs)
ints := []int{7, 2, 4}
sort.Ints(ints)
```
2. 通过实现`sort.Interface`接口的`Len、Less、Swap`方法， 就可以使用sort包的通用Sort方法，实现自定义函数排序。实现一个按字符串长度排序的例子：  
```go
type byLength []string
func (s byLength) Len() int {
    return len(s)
}
func (s byLength) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}
func (s byLength) Less(i, j int) bool {
    return len(s[i]) < len(s[j])
}

func main() {
    fruits := []string{"peach", "banana", "kiwi"}
    sort.Sort(byLength(fruits))
    fmt.Println(fruits)
}
// output:[kiwi peach banana]
```
3. 我们用`panic`来表示程序正常运行中不应该出现的错误， 或者我们不准备优雅处理的错误。当函数返回我们不知道如何处理（或不想处理）的错误值时，中止操作，这是panic常用的方法。  
4. defer一般是执行清理工作，比如关闭文件等，并且在错误检查之后，因为它代表对象已经存在，才会进行清理工作，否则会出现空指针错误。  
5. `recover`可以阻止panic中止程序，并让它继续执行，recover的返回值是在调用panic时抛出的错误。  
6. `os.Open`只读文件，`os.OpenFile`可以指定是否可读可写。  
7. `os.Args`提供原始命令行参数访问功能。  

### 高质量编程

高质量的代码有4个特性：  
- 正确性，完备考虑各种边界条件；
- 可靠性，处理各种异常、错误，保证服务的稳定；
- 易维护，逻辑简单，后续调整或添加功能能够快速支持；
- 易阅读，代码能够被他人清晰阅读，容易理解。  

高质量编程3个通用原则：  
- 简单，没有多余的复杂性，能够以清晰的逻辑编写代码；  
  复杂的程序逻辑，不易重构和优化，遇到问题也难以排查。  
- 可读，代码是给人阅读的，可读才可维护；
- 生产力，编程更多是团队合作，团队整体的工作效率是十分重要的。  
  Go通过`go fmt`统一代码格式，降低新成员上手项目代码的成本。  

#### 编码规范
**代码格式：** 使用`go fmt`自动格式化代码；  
**注释：**   
- 解释代码作用。代码实现的原因是什么，代码是如何实现的，代码什么情况会出错；  
- 对于公共符号始终要注释。比如公共的变量、常量、函数以及结构等；  
- 代码是最好的注释，注释应该提供代码来表达出的上下文信息。  

**命名规范：**   
- 变量名要简洁，缩略词全大写，比如`ServeHTTP`;  
- 变量距离被使用的地方越远，越需要携带更多的上下文信息，使其含义易读。  

下边举几个例子，感受一下变量命名的好坏。  
i和index的作用域仅限于for循环，index的额外冗余并没有增加对于程序的理解。  
```go
// Bad
for index := 0; index < len(s); index++ {
  // do something
}
// Good
for i := 0; i < len(s); i++ {
  // do something
}
```
deadline有特殊含义，替换成t降低了变量名的信息量。  
```go
// Good
func (c *Client) send(req *Request, deadline time.Time)
// Bad
func (c *Client) send(req *Request, t time.Time)
```
函数名不携带包名的上下文，因为包名和函数名总是成对出现；
```go
// http包中函数
// Good
func Serve(l net.Listener, handler Handler) error
// Bad
func ServeHTTP(l net.Listener, handler Handler) error
```

- package名只由小写字母组成；
- 简短并包含一定的上下文信息，比如schema、task等；
- 不要与标准库同名，比如不要使用sync、strings等；
- 不要使用常用变量名作为包名，比如使用bufio而不是buf；
- 包名使用单数，比如使用encoding而不是encodings；
- 谨慎使用缩写，在不破坏上下文的情况下fmt比format更简短。

**控制流程：**  
- 避免嵌套，保持正常流程清晰；
```go
// Bad
if foo {
  return x
} else {
  return nil
}
// Good
if foo {
  return x
}
return nil
```
- 尽量保持正常代码路径为最小缩进，遵守线性原理；
  也就是优先处理错误情况/特殊情况，尽早返回或继续循环来减少嵌套。  
```go
// Bad
func OneFunc() error {
  err := doSomething()
  if err == nil {
    err := doAnotherThing()
    if err == nil {
      return nil // normal case
    }
    return err
  }
  return err
}
// Good
func OneFunc() error {
  if err := doSomething(); err != nil {
    return err
  }
  if err := doAnotherThing(); err != nil {
    return err
  }
  return nil // normal case
}
```

**错误和异常处理：**  
- 简单错误是指仅出现一次的错误，在其他地方不需要捕获该错误；
  优先使用`errors.New`来创建匿名变量直接表示简单错误；  
  如果有格式化的需求，使用`fmt.Errorf`。  
```go
func defaultCheckRedirect(req *Request, via []*Request) error {
  if len(via) >= 10 {
    return errors.New("stopped after 10 redirects")
  }
  return nil
}
```
- 错误的Wrap提供了一个error嵌套另一个error，生成了一个error跟踪链；
  在`fmt.Errorf`中使用`%w`将一个错误关联至错误链中。  
```go
list, _, err := c.GetBytes(cache.Subkey(a.actionID, "srcfiles"))
if err != nil {
  return fmt.Errorf("reading srcfiles list:%w", err)
}
```
- 使用`errors.Is`判定一个错误是否为特定错误；
  不同于`==`，使用该方法可以判定错误链上的所有错误是否含有特定的错误。  
```go
data, err = lockedfile.Read(targ)
if errors.Is(err, fs.ErrNotExist) {
  return []byte{}, nil
}
return data, err
```
- 在错误链上获取特定种类的错误，使用`errors.As`；
  它和Is的区别在于，会提取出调用链中指定类型的错误，并将错误赋值给定义好的变量，以便后续处理，下例是将问题的patch打印出来：  
```go
if _, err := os.Open("non-existing"); err != nil {
  var pathError *fs.PathError
  if errors.As(err, &pathError) {
    fmt.Println("Failed at path:", pathError.Path)
  } else {
    fmt.Println(err)
  }
}
```
- 当程序启动阶段发生不可逆转的错误时，可以在init或main使用panic，其他业务代码不建议使用；
- recover只能在被defer的函数中使用，嵌套无法生效，只在当前goroutine生效；
```go
func (s *ss) Token(skipSpace bool, f func(rune) bool) (tok []byte, err error) {
  // defer LIFO
  defer func() {
    if e := recover(); e != nil {
      if se, ok := e.(scanError); ok {
        err = se.err
      } else {
        panic(e)
      }
    }
  }()
  // do something
}
```

#### 性能优化建议
性能优化的前提是满足程序的正确可靠，简洁清晰。性能优化是综合评估，时间效率和空间效率常常是对立的两方，对Go语言特性，总结其性能优化建议。  
**Benchmark：**  
使用Go自带的性能评估工具，以实际数据评价代码性能，示例：  
```go
// fib.go
func Fib(n int) int {
  if n < 2 {
    return n
  }
  return Fib(n-1)+Fib(n-2)
}
// fib_test.go
func BenchmarkFib10(b *testing.B) {
  for n := 0; n < b.N; n++ {
    Fib(10)
  }
}
```
命令：`go test -bench=. -benchmem`  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205311436582.png)  
`BenchmarkFib10-8`：测试函数名-8，8是GOMAXPROCS的值，默认为CPU核数；  
`4473883`：b.N的值，表示一共执行这么多次；   
`247.4 ns/op`：表示每次执行花费的时间；  
`0 B/op`：表示每次执行申请多大的内存；  
`0 allocs/op`：表示每次执行申请几次内存。  

**Slice：**
- 预分配：  
  在使用make初始化slice时预分配内存，提供容量信息，示例对比：  
```go
func NoPreAlloc(size int) {
  data := make([]int, 0)
  for k := 0; k < size; k++ {
    data = append(data, k)
  }
}
func PreAlloc(size int) {
  data := make([]int, 0, size)
  for k := 0; k < size; k++ {
    data = append(data, k)
  }
}
```
命令：`go test -bench=. -benchmem`  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205311500324.png)  
slice预分配内存不仅节约了时间，内存申请次数也只有一次。  
出现这种现象的原因是，slice是数组片段的描述，包括：数组指针，数组长度，数组容量；切片操作并不复制切片指向的元素，创建一个新切片会复用原来切片的底层数组。  
```go
type slice struct {
  array unsafe.Pointer
  len int
  cap int
}
```
`append`有两种情况：  
1. append之后，slice长度小于等于cap，直接利用原底层数组剩余空间；  
2. append之后，slice长度大于cap，会分配一块更大的区域容纳新的底层数组。  

为了避免发生内存拷贝，如果知道最终slice大小，预分配cap能够减少内存分配，获得更好的性能。  
- 大内存未释放：  
  如果原slice是由大量元素组成，我们在此基础再切片，只使用了一小部分，但底层数组仍然在内存中占用大量空间，得不到释放。可以使用copy代替re-slice，示例如下，完整代码见[Github](https://github.com/KevinChens/youth-camp/tree/main/lesson3/copydemo)：  
```go
func GetLastBySlice(origin []int) []int {
  return origin[len(origin)-2:]
}
func GetLastByCopy(origin []int) []int {
  result := make([]int, 2)
  copy(result, origin[len(origin-2):])
  return result
}
```
命令：`go test -run=. -v`  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205311550828.png)  

**Map：**  
同样的map也应该预分配内存，向map添加元素的操作会触发map的扩容机制，提前分配空间可以减少内存拷贝和Rehash的消耗。  

**String：**  
字符串的拼接方式有`+`、`strings.Builder`和`bytes.Buffer`，三者性能表现如下：  
```go
func Plus(n int, str string) string {
  s := ""
  for i := 0; i < n; i++{
    s += str
  }
  return s
}
func StrBuilder(n int, str string) string {
  var builder strings.Builder
  for i := 0; i < n; i++ {
    builder.WriteString(str)
  }
  return builder.String()
}
func ByteBuffer(n int, str string) string {
  buf := new(bytes.Buffer)
  for i := 0; i < n; i++ {
    buf.WriteString(str)
  }
  return buf.String()
}
// n=10000, str="hello"
```
命令：`go test -bench=. -benchmem`  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205311619238.png)  
分析：  
- 使用`+`拼接性能最差，后两者性能接近，但`strings.Buffer`更快。  
- 字符串在Golang中是不可变类型，占用内存大小是固定的；
- `+`拼接是不断开辟一个新空间，每次操作都会重新分配内存；
- `strings.Builder`和`bytes.Buffer`底层都是`[]byte`数组；
- 内存扩容策略，不需要每次拼接重新分配内存。  

为什么`strings.Builder`比`bytes.Buffer`更快：  
```go
func (b *Buffer) String() string {
  if b == nil {
    return "<nil>"
  }
  return string(b.buf[b.off:])
}
func (b *Builder) String() string {
  return *(*string)(unsafe.Pointer(&b.buf))
}
```
`bytes.Buffer`转化为字符串会重新申请一块空间，`strings.Builder`直接将底层`[]byte`转换成字符串类型返回，因此更快。  
如果想要更高效的字符串构造方法，可以使用预分配+`strings.Builder`。  
```go
func PreStrBuilder(n int, str string) string {
  var builder strings.Builder
  builder.Grow(n*len(str))
  for i := 0; i < n; i++ {
    builder.WriteString(str)
  }
  return builder.String()
}
func PreByteBuffer(n int, str string) string {
  buf := new(bytes.Buffer)
  buf.Grow(n*len(str))
  for i := 0; i < n; i++ {
    buf.WriteString(str)
  }
  return buf.String()
}
```
命令：`go test -bench=. -benchmen`  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205311648839.png)  
`strings.Builder`只有一次内存分配，`bytes.Buffer`有两次内存分配。  

**空结构体：**  
空结构体`struct{}`实例不占据内存空间，可为占位符。  

**atomic包：**  
在遇到多线程场景时，比如实现一个多线程共用计数器，如何保证计数准确性，线程安全。  
可以使用atomic包的原子操作，也可以使用互斥锁，示例如下：  
```go
type atomicCounter struct {
  i int32
}
func AtomicAddOne(c *atomicCounter) {
  atomic.AddInt32(&c.i, 1)
}
type mutexCounter struct {
  i int32
  m sync.Mutex
}
func MutexAddOne(c *mutexCounter) {
 c.m.Lock()
 c.i++
 c.m.Unlock()
}
```
命令：`go test -bench=. -benchmen`  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205311716942.png)  
锁的实现是通过操作系统来实现的，是系统调用；atomic操作是通过硬件实现，效率比锁高；  
`sync.Mutex`一般用来保护一段逻辑，不仅仅是保护一个变量；  
对于非数值操作，可以使用`atomic.Value`，能承载一个`interface{}`。  

### 性能调优实战
性能调优原则：  
- 依靠数据而不是猜测；
- 定位最大瓶颈而不是细枝末节；
- 不要过早优化；
- 不要过度优化。  

#### pprof实战
pprof是用于可视化和分析性能，分析数据的工具，能够告知程序在什么地方耗费了多少CPU，多少内存等实际的数据指标。  
**pprof功能**  
  ![img](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205312151752.png)  
pprof可以采样程序运行时的CPU、堆内存、goroutine、锁竞争、阻塞调用和系统线程等的使用数据，同时通过列表、调用图、火焰图、源码以及反汇编等视图展示采集到的性能指标。  

**pprof排查**  
1. 搭建项目，git clone[源码](https://github.com/wolfogre/go-pprof-practice)，项目提前埋入炸弹代码，能够产生可观测的性能问题。会占用1CPU核心和超过1GB的内存。  
```go
func main() {
  ...
  runtime.GOMAXPROCS(1)              // 限制CPU使用数
  runtime.SetMutexProfileFraction(1) // 开启锁调用跟踪
  runtime.SetBlockProfileRate(1)     // 开启阻塞调用跟踪
  go func() {
    // 启动http server
    if err := http.ListenAndServe(":6060", nil); err != nil {
      log.Fatal(err)
    }
    os.Exit(0)
  }()
  ...
}
```
`main.go`初始化http服务和pprof接口，`import _ "net/http/pprof"`，它会注册pprof的handler到http server。  
2. 浏览器查看指标，我们可以在浏览器打开`http://127.0.01:6060/debug/pprof`查看基本的性能统计。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205312217729.png)  
  `allocs:` 内存分配情况；  
  `blocks:` 阻塞操作情况；  
  `cmdline:` 当前程序命令行调用情况；  
  `goroutine:` 当前所有goroutine的堆栈信息；  
  `heap:` 堆上内存使用情况（同allocs）；  
  `mutex:` 锁竞争操作情况；  
  `profile:` CPU占用情况；  
  `threadcreate:` 当前所有创建的系统线程的堆栈信息；  
  `trace:` 程序运行跟踪信息。  

浏览器数据可读性比较差，可借助pprof阅读这些数据指标。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205312226921.png)  
3. CPU问题排查；  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205312229843.png)  
  明显CPU占用异常，pprof的采样结果是将一段时间内的信息汇总输出到文件中，因此要拿到这个profile文件，可使用pprof工具连接接口下载需要的数据。我们使用`go tool pprof`+采样链接启动采样。  
  `go tool pprof "http://127.0.0.1:6060/debug/pprof/profile?seconds=10"`，profile代表采样对象是CPU使用情况，如果在浏览器打开该链接，会启动一个60s的采样，并在结束后下载文件，我们这里加上seconds=10，让其采样10s。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205312248005.png)  
  输入`top`可以查看CPU占用最高的函数：  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205312303347.png)  
  `flat` 表示当前函数本身的执行耗时；  
  `flat%` 表示flat占CPU总量的比例；  
  `sum%` 表示上面所有行的flat%总和；  
  `cum` 表示当前函数加上其调用函数的总占用；  
  `cum%` cum占总量的比例。  
  表格前面描述了采样的总体信息，默认展示占用资源最高的10个函数，如果需要看最高的N个，输入topN，比如top3查看前3个占用最高的调用。  
  可以看出表格第一行是问题所在，这个函数本身执行耗时flat，调用函数总耗时cum都3s左右，但flat%和cum%占用都超过90%。  
  从表中还能看到一些其他情况，比如flat和cum相等，或者有时为0。  
- 什么情况下，falt==cum？函数中没有对其他函数的调用。  
- 什么情况下，flat==0？函数中只有其他函数的调用。  

输入`list Eat`查找这个函数，list会根据给定的正则表达式查找代码，并按行展示出每一行的占用。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205312313422.png)  
可以看见这条3.15s的占用是问题所在，定位成功。  
输入`web`，生成一张调用关系图，调用关系可视化。可能需要安装graphviz组件，通过命令`brew install graphviz`安装即可。其中方框最红最大的就是占用CPU最多的函数。CPU炸弹定位完成，输入`q`退出终端。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205312330530.png)  
为了后续的展示，将问题代码注释，拆除这个炸弹，重新运行程序。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205312341785.png)  
明显CPU占用已经降下来，但内存占用问题还存在。  

4. Heap堆内存问题排查；  
  这里我们通过`-http=:8080`开启pprof自带的Web UI，使用命令`go tool pprof -http=:8080 "http://127.0.0.1:6060/debug/pprof/heap"`完成对heap的采样。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205312351779.png)  
  明显看出，Steal函数是问题所在，占用了1.2GB内存。通过view菜单可以看不同视图比如top视图，source视图，可搜索Steal进行函数定位：  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205312355351.png)  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202205312356711.png)  
  问题函数已定位，将其注释，发现内存正常，至此两个炸弹已拆除。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206010001899.png)  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206010013343.png)  
  采样也没有异常节点，但内存问题就全部排除了吗？注意右上角`unknow_inuse_space`，打开sample菜单，发现堆内存有4个指标：  
- alloc_objects: 程序累计申请的对象数；
- alloc_space: 程序累计的内存大小；
- inuse_objects: 程序当前持有的对象数；
- inuse_space: 程序当前占用的内存大小。  

默认展示`inuse_space`，只展示当前持有的内存，但如果有的内存已经释放，inuse采样就不会展示了，切换到`alloc_space`指标，继续`alloc`内存问题分析（左：Source视图Run函数；右：Top视图）：  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206010017480.png)  
明显Run函数是问题所在，它每次申请16MB大小内存，累计申请超过2.69GB，注释问题代码，继续后续排查，至此内存炸弹已经全部拆除。  

5. goroutine泄漏，导致内存泄漏；  
  执行命令`go tool pprof -http=:8080 "http://127.0.0.1:6060/debug/pprof/goroutine"`，可以看见一张很长的调用关系图，节点过多，难以阅读，可以打开view菜单，通过Flame Graph 火焰图阅读。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206010035686.png)  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206010041323.png)  
  由上到下表示调用顺序，每一块代表一个函数，越长代表占用CPU的时间更长，火焰图是动态的，支持点击块进行分析，可以看见Drink函数调用创建了超过95%的goroutine。  
  在source视图可见问题函数，每次执行10条无意义的goroutine，每条等待30s后退出，导致goroutine泄漏。  
  设想一下，如果发起的goroutine没有退出，同时不断有goroutine被启动，对应的内存占用持续增长，CPU调度压力不断增大，最终进程会被系统kill。注释问题代码，goroutine泄漏问题解决。  

6. mutex，锁竞争；  
  执行命令`go tool pprof -http=:8080 "http://127.0.0.1:6060/debug/pprof/mutex"`，在Graph视图能够定位到问题函数Howl，在Source视图能够看见是哪一行代码发生了锁竞争。在这个函数中，goroutine等待了1s才解锁，阻塞了，注释即可。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206010721971.png)  

7. block阻塞；  
执行命令`go tool pprof -http=:8080 "http://127.0.0.1:6060/debug/pprof/block"`，除了锁竞争会导致阻塞，其他逻辑，比如读取一个channel也会导致阻塞，在页面中可以看到阻塞操作还有两个。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206010738053.png)  
从Graph视图到Source视图，看到Pee函数读取了一个`time.After()`生成的channel，导致了这个goroutine阻塞了1s而不是等待了1s，注释代码解决阻塞。  
通过上述操作只定位到了1个block，终端输入`go tool pprof "http://127.0.0.1:6060/debug/pprof/block"`和`top`可以看见另外一个阻塞操作的节点。有3个节点因为`cumulative`小于4.24s被drop，它的总用时小于总时长的千分之五，所以被省略了，没有展示。这样的过滤策略能够更加有效地突出问题所在，省略相对没有问题的信息，便于问题定位，解决主要瓶颈问题。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206010758578.png)  
那如何查看另外一个阻塞操作呢？可以通过暴露的接口地址直接访问，打开block，发现第二个阻塞操作发生在`http handler`。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206010751875.png)  
至此代码中所有炸弹已经找出并解决，我们完成了以下实践操作：  
- 5种使用pprof采集的常用性能指标：CPU、堆内存、Goroutine、锁竞争和阻塞；
- 2种展示方式：交互式终端和Web UI；
- 4种视图：Top、Graph、Source和Flame Graph。  
  
#### 采样过程和原理 
知其然，知其所以然，看看pprof内部实现是怎样的。  
首先是**CPU**，有3个主要采样指标：  
- 采样对象：所有调用栈和它们的占用时间；
- 采样率：100次/秒，固定值；
进程每秒暂停100次，记录当前的调用栈信息，汇总后，根据调用栈在采样中出现次数推断函数的运行时间；  
- 采样时间：从手动启动到手动结束。  
这个100次/秒的定时暂停机制在unix或类unix系统上是通过依赖信号机制实现的。每次暂停都会接收到一个信号，通过系统计时器来保证这个信号固定频率发送。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206010815570.png)  
采样具体流程：  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206010901466.png)  
一共3个对象：  
- 操作系统：每10ms向进程发送一次SIGPROF信号；
- 进程：每次接受到SIGPROF会记录调用堆栈；
- 写缓冲：每100ms读取已经记录的调用栈并写入输出流。

第二个是**堆内存**，由于pprof的局限性，采样的是堆内存而不是内存，内存采样在实现上依赖内存分配器的记录，所以只能记录在堆上分配，会参与GC的内存，一些其他的内存分配，比如调用结束就会回收的栈内存，更底层使用的cgo调用分配的内存等，这些是不会被内存采样记录的。其采样过程和各个指标如下：  
- 采样程序通过内存分配器在堆上分配和释放内存，记录分配/释放的大小和数量；
- 采样率是每分配51KB记录一次，可以在运行开头修改，设为1则每次分配都会记录；
- 采样时间是从程序运行开始就一直采样；
和CPU、Goroutine不同，内存的采样是一个持续过程，会记录从程序运行开始所有分配或释放的内存大小和对象数量，并在采样时遍历这些结果并进行汇总。  
- 采样指标有`alloc_space,alloc_objects,inuse_space,inuse_objects`；
- inuse=alloc-free；
alloc是指从程序运行开始的累计指标，inuse是指通过累计分配减去累计释放得到的程序当前持有的指标，可以通过两次alloc的差值得到某一段时间程序分配的内存大小和数量。  

第三个是**goroutine和系统线程**的采样，它们在实现上非常相似，都是在STW之后，遍历所有goroutine/所有线程的列表，并输出堆栈，最后`Start The World`继续运行；这个采样是立刻触发的全量记录，可以通过比较两个时间点的差值来得到某一段时间段的指标。  
- goroutine：记录所有用户发起且在运行中的goroutine，是入口非runtime开头的goroutine，以及runtime.main所在goroutine的信息和创建这些goroutine的调用栈；
- threadcreate：记录程序创建的所有系统线程的信息。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206010924984.png)  
  
最后是**block阻塞和锁竞争**的采样，它们的采样记录都是对应操作发生的调用栈、次数和耗时，但指标的采样率含义不同，但实现上是基本相同的，都是一个主动上报的过程。  
- 阻塞操作的采样率是一个阈值，消耗超过阈值时间的阻塞操作才会被记录，1为每次操作都会记录；  
- 锁竞争的采样率是一个比例，运行时会通过随机数来只记录固定比例的锁操作，1为每次操作都会记录；
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206010934245.png)  
在阻塞操作或锁操作发生时，会计算出消耗的时间，连同调用栈一起主动上报给采样器，采样器根据采样率可能会丢弃一些记录。在采样时，采样器会遍历已经记录的信息，统计具体操作的次数、调用栈和总耗时。和堆内存一样，可以对比两个时间点的差值计算出一段时间内的操作指标。  

至此，整个pprof实战、常用指标和它们实现的原理有了初步了解。这次实战只是模拟的一个pprof性能分析的小场景，但排查思路是通用的，对于解决性能问题和性能调优是十分有帮助的。  

#### 案例
接下来了解在工程中进行性能调优的实际案例。程序优化从不同的应用层次，分为业务服务优化、基础库优化和Go语言优化。  
- 业务服务：直接提供功能的程序，比如专门处理用户评论操作的程序；
- 基础库：提供通用功能的程序，主要是针对业务服务提供功能，比如监控组件，负责收集业务服务的运行指标；
- Go语言本身的优化。  

以下是系统部署简单示意图，客户端请求经过网关转发，由不同的业务服务处理，而业务服务可能依赖其他的服务，也可能依赖存储、消息队列等组件。  
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206011358378.png)  
- 服务：能单独部署，承载一定功能的程序；
- 依赖：Service A的功能实现依赖Service B的响应结果，Service B就是Service A的依赖；
- 调用链路：支持一个接口请求的相关服务集合及其相互之间的依赖关系；
- 基础库：公共的工具包、中间件。

业务服务优化主要流程：  
- 建立服务性能评估手段；
- 分析性能数据，定位性能瓶颈；
用pprof采样性能数据，分析服务表现；
- 重点优化项改造；
发现性能瓶颈，进行服务改造，重构代码，使用更高效的组件；
- 优化效果验证；
通过压测对比和正确性验证之后，服务可以上线进行实际收益评估。

整个流程循环并行执行，每个优化点可能不同，可以分别评估验证。

**建立服务性能评估手段：**  
- 服务性能评估方式：  
单独benchmark无法满足复杂逻辑分析；  
不同负载情况下性能表现差异；  
- 请求流量构造：  
不同请求参数覆盖逻辑不同；  
尽量模拟真实流量情况，分析真正性能瓶颈；  
- 压测范围：单机压测和集群压测；  
- 性能数据采集：单机性能数据和集群性能数据；  
录制线上的请求流量，通过控制回放速度来对服务进行测试，测试范围可以是单个实例，也可以是整个集群，同样性能采集也会区分单机和集群。  

服务性能评估手段的产出是一个服务的性能指标分析报告，比如压测报告会统计压测期间服务的各项监控指标，包括qps，延迟等内容；同时在压测过程中，可以采集服务的pprof数据，使用刚刚实践的方式分析性能问题。  

**分析性能数据，定位性能瓶颈：**  
- 使用库不规范；  
业务服务常见的性能问题可能是使用基础组件不规范，比如每次使用配置都会进行json解析，拿到配置项，实际组件内部提供了缓存机制，只有数据变更的时候才会重新解析json；  
又比如日志使用不规范，一部分是调优日志发布到线上，一部分是线上服务在不同的调用链路上数据有差别，测试场景日志量能够接收，但到了真实线上全景场景，会导致日志量增加，影响性能。  
- 高并发场景优化不足；  
服务有高峰期和底峰期，监控组件的CPU资源等占用变化较大，主要原因是监控数据上报是同步请求，在请求量上涨，监控打点数据量增加，达到性能瓶颈，造成阻塞，影响业务逻辑的处理，后续改成异步上报提升性能。  

**重点优化项改造：**  
定位性能瓶颈后，也进行了对应的修复手段，能直接发布上线吗？  
性能优化的前提是保证**正确性**，所以在变动较大的性能优化上线之前，需要进行正确性验证。由于线上的场景和流程太多，要借助自动化手段保证优化后程序的正确性。  
同样是线上请求录制，这里不仅包含请求参数录制，还会录制线上的返回内容，重放时对比线上的返回内容和优化后服务的返回内容进行正确性验证。  

**优化效果验证：**  
- 重复压测验证；
查看是否达到期望的优化，记录真正的优化效果；  
同时压测不能保证和线上表现完全一致，有时还要通过线上的表现再进行分析改进，是一个长期的过程；  
- 上线评估优化结果；
关注服务监控，逐步放量，收集性能数据；  

**进一步优化，服务整体链路分析：**  
以上优化都是针对单个服务的优化过程，从更高的视角，查看是否还有性能优化空间；  
- 规范上游服务调用接口，明确场景需求；
- 分析链路，通过业务流程优化提升服务性能；
  ![](https://raw.githubusercontent.com/KevinChens/blog-image-bed/main/imgs/202206011358378.png)   
在熟悉服务的整体部署情况以后，可以针对具体的接口链路进行分析调优；比如Service A调用Service B是否存在重复调用的情况，调用时是否更小的结果数据集就能满足需求，接口是否一定要实时数据，能否在Service A进行缓存，减轻调用压力等等。  
这种优化需要结合特定业务场景，但能合理利用资源。  
  
适用范围更广的是基础库的优化。比如在实际业务服务中，为了评估某些功能上线后的效果，需要进行AB实验（一种验证假设方法，核心方法以及原理分别是对照实验和假设检验），查看不同策略对核心指标的影响，很多服务就会使用AB实验的SDK，如果优化AB组件库的性能，那用到的服务都会有性能提升。  
优化过程：  
- 分析基础库核心逻辑和性能瓶颈；  
设计完善改造方案，数据按需获取，数据序列化协议优化；  
- 内部压测验证；
- 推广业务服务落地验证。  

最基础的是编译器和runtime的优化。这些优化对于业务服务接入十分简单，只需要调整编译配置，通用性很强。 
优化过程：  
- 优化内存分配策略；
- 优化代码编译流程，生成更高效的程序；
- 内部压测验证；
- 推广业务服务落地验证。

### 总结
至此，整个高质量编程与性能调优实战课程总结归纳完毕，在这门课学到了很多。对于实际工作中服务性能优化有所了解，算是入个门不过分吧；对于如何分析性能问题，服务优化的原则和优化流程都进行了学习。同时也对性能分析工具pprof的基本原理，相关使用操作和如何排查性能问题，进行了简单的入门实践。  
收获很多，十分感谢张雷老师的授课讲解！  

### 引用参考
1. [Go by Example 中文版](https://gobyexample-cn.github.io)
2. [Golang标准库文档](https://studygolang.com/pkgdoc)
3. [极客兔兔：大量内存得不到释放](https://geektutu.com/post/hpg-slice.html)
4. [Go pprof实战](https://github.com/wolfogre/go-pprof-practice)



