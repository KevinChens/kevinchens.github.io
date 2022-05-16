# Golang基础语法


## Golang基础语法 | 青训笔记

这是我参与「第三届青训营 -后端场」笔记创作活动的的第1篇笔记。  

从Go语言特性入手，讲述了Golang区别于其他语言的优点，通过Go by Example对基础语法进行了巩固学习。  
然后对Golang常用标准库的使用和注意事项通过代码示例，最后是命令行词典和socks5代理两个小项目的实践，进一步了解Golang。  

### Go语言特性

- 高并发，高性能  
  Golang是基于其他语言的痛点，开发的新时代语言。  
  天生支持高并发，能够充分利用多核CPU，高性能开发简单。  
- 简单易学  
- 标准库，工具链丰富  
- 静态链接，快速编译  
- 跨平台  
  多平台，多系统，可交叉编译，无需配置编译环境。  
- 垃圾回收  
  自动管理内存释放，专注业务逻辑编程。  

**应用场景**  
- 公司  
  使用Golang开发的公司越来越多，比如国内的ByteDance，Tencent，美团，七牛云等，国外的Google，Facebook等。 
- 领域  
  Golang在云计算，微服务，大数据，区块链，物联网等领域都有广泛的使用，其中在云计算，微服务领域有非常高的市场占有率。 
- 云原生组件  
  Docker，Kubernetes，etcd，Prometheus等全是通过Golang实现。

### 重点语法

通过Go by Example进行入门上手，对重点语法进行总结归纳。下面展示核心代码部分，完整源码，见[Github仓库](https://github.com/KevinChens/youth-camp/tree/main/lesson1/go-by-example)  
#### 数据类型
1. string支持`+`拼接，也可以通过`fmt.Sprintf()`，`strings.Join()`和`buffer.WriteString()`，`build.WriteString()`进行拼接。
```go
str1, str2 := "go", "lang"
fmt.Println(str1 + str2)
fmt.Println(fmt.Sprintf("%s%s", str1, str2))
fmt.Println(strings.Join([]string{str1, str2}, ""))

var bt bytes.Buffer
bt.WriteString(str1)
bt.WriteString(str2)
fmt.Println(bt.String())
fmt.Println("go" + "lang")

var build strings.Builder
build.WriteString(str1)
build.WriteString(str2)
fmt.Println(build.String())
```
2. slice比array更常用，slice可以组成多维数据结构，内部长度可以不一致。slice可以模拟`stack`，`queue`。
3. map的键值对是无序的，可以自行按照`key`或`value`进行排序。检测key是否存在：
```go
m := make(map[string]int)
if v, ok := m["zhangsan"]; ok {
  fmt.Println("key exists, value:", v)
} 
```
4. range可对slice，map，string进行遍历，第一个参数是index(key)，第二个参数是value。
```go
slice := []int{1, 3, 4, 6}
for i, v := range slice {
  fmt.Println(i, v)
}
```
5. Go支持指针，允许通过引用传递来传递值和修改值。  
  值传递是数据的拷贝，会分配新的内存存储；  
  引用传递是指针的拷贝，共享数据的内存地址，通常使用指针是为了**修改值或节约内存空间**。  
6. string等价于`[]byte`，Golang中字符是`rune`类型（uint8）, 用单引号括起来的值。  
7. struct是带类型的字段集合，是可变的，通过构造函数封装创建结构体实例是习惯用法。 
```go
type User struct {
  Name string
  Age int
}
func Constructor() User {
  return User{Name: "zhangsan", Age: "22"}
}
```


#### 函数 
1. 对于多返回值函数，可用空白标识符`_`仅返回需要的值：
```go
func calc() (sum, sub int) {
  return 1+2, 1-2
}

sum, _ := calc()
```
2. 变参函数的使用：
```go
func sum(nums ...int) int {
  ans := 0
  for _, num := range nums {
    ans += num
  }
  return ans
}

ans := sum(1, 2)
ans = sum(1, 2, 3)
// 也可以传slice
nums := []int{1, 2, 3, 4}
ans = sum(nums...)
```
3. interface是方法签名的集合，在Go中实现一个interface就是实现其中所有方法。  
   如果一个变量实现了接口，就可以调用接口的方法。
4. 可以为值类型或指针类型的接收者定义方法，想避免调用方法时产生数据拷贝或想修改接收者的值，可以选用指针来调用方法。  
```go
type User struct {
  Name string
  Age int
}

func (u User) ModifyInfo1() User {
  u.Name = "lisi"
  u.Age = 15
  // u修改不成功
  return u
}

func (u *User) ModifyInfo2() *User {
  u.Name = "lisi"
  u.Age = 15
  // u修改成功
  return u
}
```
5. Go习惯用独立的，明确的返回值来传递错误信息。函数的最后一个返回值常常是error类型的错误，`errors.New`使用给定的错误信息构造一个基本的error值。也可以使用Error()方法，自定义error类型。  
  在if的同一行进行错误检测是Go常用的方式。如果想在程序中使用自定义错误类型的数据，需要通过类型断言得到这个自定义错误类型的实例。  

### 常用标准库
#### fmt
输出：`PrintXXX`, `FprintXXX`, `SprintXXX`, `ErrorfXXX`  
```go
fmt.Print("不换行")
str := "golang"
fmt.Printf("%s\n", str)
fmt.Println("有换行")
// Fprint将内容输出到io.Writer类型的变量中，常用在写入内容到文件
fileObj, err := os.OpenFile("./xx.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
if err != nil {
  fmt.Println("打开文件错误，err:", err)
}
fmt.Fprintf(fielObj, "向文件中写入：%s", str)
fmt.Fprintln(os.Stdout, "向标准输出写入内容")
// Sprint把传入的数据生成并返回一个字符串
s1 := fmt.Sprint("golang")
s2 := fmt.Sprintf("s1:%s", s1)
s3 := fmt.Sprtinln("golang有换行")
// Errof根据format参数生成格式化字符串并返回一个包含该字符串的错误, 通常用来自定义错误类型
err := fmt.Errof("这是一个自定义错误类型")
```
输入：`ScanXXX` 
```go
var name string
var age int
fmt.Scan(&name, &age)
fmt.Scanf("%s, %d\n", &name, &age)
fmt.Scanln(&name, &age)  // 遇到换行才停止
// 如果想要获取包含空格等完整输入内容，可以用bufio.NewReader
reader := bufio.NewReader(os.Stdin)
text, _ := reader.ReadString("\n")
text = strings.TrimSpace(text)
fmt.Printf("%#v\n", text)
```
输入还有`FscanXXX`从`io.Reader`读取和`ScanXXX`从指定字符串读取。  
#### time
时间戳：  
```go
now := time.Now()
timestamp1 := now.Unix()  // 时间戳
timestamp2 := now.UnixNano()  // 纳秒级时间戳
```
时间间隔：`time.Duration`表示1ns，`time.Second`表示1s
时间格式化，以Go的诞生时间固定格式模板：  
```go
now := time.Now()
fmt.Println(now.Format("2006-01-02 15:04:05.000 Mon Jan"))
```
#### strconv
string和int的转换：  
```go
num := strconv.Atoi("123")
str := strconv.Itoa(num)
```
`ParseXXX`将字符串转换为给定类型：  
```go
boolType, err := strconv.ParseBool("true")
intType, err := strconv.ParseInt("23", 10, 64)  // 10进制，int64
floatType, err := strconv.ParseFloat("3.14", 64)  // float64
uintType, err := strconv.ParseUint("23", 64)  // uint64
```
`FormatXX`将给定类型转换为string。  
#### strings
包括了操作字符的常用函数，比如大小写转换，是否有指定前缀或后缀，字符串拼接或分离等。   
```go
str := "golang"
strings.Contains(str, "lang")
strings.HasPrefix(str, "go")
strings.ToUpper(str)
strings.Replace(str, "g", "G", -1)
```
#### encoding/json
这个包实现了对json的编码和解码。  
```go
// 序列化，其他数据类型转json
type User struct {
  Name string `json:"name"`
  Age int     `json:"age"`
}
u := User{Name:"zhangsan", Age:"18"}
buf, err := json.Marshal(u)
if err != nil {
  panic(err)
}
fmt.Println(string(buf))
// 反序列化，json转其他数据类型
var u2 User
err = json.Unmarshal(buf, &u2)
if err != nil {
  panic(err)
}
fmt.Printf("%#v\n", u2)
```

### 实战练习
#### 命令行词典
需求：调用第三方翻译API实现一个命令行词典，使用格式：`simpleDict word`  
分析：整个过程主要是三个步骤：发送HTTP请求，解析响应json，打印出结果。
1. 抓包，打开[彩云小译](https://fanyi.caiyunapp.com/#/)，按F12，输入你要翻译的单词，比如`translate`, 可在DevTools的Network查看HTTP的post请求信息：  
  ![img](https://s2.loli.net/2022/05/16/SpN2MmcEPJCotjH.jpg)  
2. 生成代码，右键`copy as curl(bash)`，复制curl命令。在终端可以返回这个请求的`json`代码，我们直接使用在线[curl2go](https://curlconverter.com/)生成该请求的Golang代码, 运行该代码同样能得到请求的`json`代码，也就是服务器返回给我们的响应体信息：  
  ![img](https://s2.loli.net/2022/05/16/OD7qSxUgzCRsiJ5.jpg)  
3. 从代码可以知道我们需要构造相应的request和response结构体，response结构体可以通过[oktools](https://oktools.net/json2go)在线生成，只需将DevTools中response的`json`复制粘贴，转换嵌套即可：  
   ![img](https://s2.loli.net/2022/05/16/I8HtzjJFiSOuglU.jpg)  
4. 从结构体当中找到我们需要返回打印的指定字段值，完善优化修改代码即可。
5. 同理，对[火山翻译](https://translate.volcengine.com/translate)和[有道智云](https://ai.youdao.com/product-fanyi-text.s)两个API进行Golang代码生成，其他第三方API也是同样的道理，只不过可能request和response的相关信息不好找或者经过Hash等处理。
6. 最后实现了三方引擎并行的一个命令行翻译词典，运行结果如下图，[完整源码](https://github.com/KevinChens/youth-camp/tree/main/lesson1/simpleDict)：  
  ![img](https://s2.loli.net/2022/05/16/kb2U7vhLzSNAjlY.jpg)  
  ![img](https://s2.loli.net/2022/05/16/dzwpgkCqR4oPuac.jpg)  

#### proxy
socks5代理协议，是明文传输，它的用途主要是作为一道屏障隔绝内部网络与互联网，并且使两者联系起来。比如某些企业的内网为确保安全性，有严格的防火墙策略，带来的副作用就是访问某些资源很麻烦。socks5相当于在防火墙开了一个口子，让授权的用户，可以通过单个端口去访问内部的所有资源。  
同样地，网络爬虫在爬取过程中，很容易遇到IP访问频率超过限制，为了突破限制，人们就会去网上找代理IP池，这些代理IP池里面的很多协议就是socks5协议。  
socks5代理过程：  
![img](https://s2.loli.net/2022/05/16/lZ6FK4GPr5vyAHi.jpg)  
主要分为4个阶段：
1. 握手阶段：  
  浏览器向socks5代理发送请求，包的内容包括了一个协议的版本号，支持的认证种类，socks5服务器会选中一个认证方式，返回给浏览器，如果返回的是00就代表不需要认证，返回其他类型会开始认证流程，对认证流程不再详述；  
2. 认证阶段：  
  认证阶段作为协商的一个子流程，它不是必须的。socks5服务器可以决定是否需要认证，如果不需要认证，那么认证阶段会被直接略过。  
  如果需要认证，客户端向socks5服务器发起一个认证请求，包括：版本， 用户名长度，对应用户名的字节数据，密码长度，密码对应的数据。  
  socks5服务器收到客户端的认证请求后，解析内容，验证信息是否合法，然后给客户端响应结果，包括：版本，状态，如果STATUS字段为0x00表示认证成功，其他的值为认证失败。当客户端收到认证失败的响应后，它将会断开连接。  
3. 请求阶段：  
  认证通过以后，浏览器会向socks5服务器发起请求，主要信息包括版本号，请求的类型，主要是connection请求，代表代理服务器要和某个域名或某个IP地址某个端口建立TCP连接，代理服务器收到响应后，会真正和后端服务器建立连接，返回一个响应；  
4. relay阶段：  
  浏览器正常发送请求，代理服务器收到请求，将请求转换到真正的服务器，真的服务器返回响应，也会给代理服务器转发给浏览器，代理服务器不关心流量的细节，可以是HTTP流量，也可以是TCP流量，这就socks5的工作原理。  
  需求：启动程序后，通过浏览器配置代理，代理服务器的日志会打印出你访问的网站的域名或者IP，也就是你的网络流量都会经过这个代理服务器。  
  分析：  
- tcp echo server  
  服务端监听你的信息并将其打印。  
- auth  
  浏览器会给代理服务器发送一个包，包括三个字段version，methods，methods的编码，代理服务器需要返回一个response，包括两个字段version，method。  
- request  
  读取携带的URL或IP+端口的包，并把它打印。  
  请求包，包括6个字段，version，command，RSV，atype，addr，port。  
- relay  
  用`net.dial`建立一个TCP连接，并启动两个goroutine，通过`io.copy`建立浏览器和下游服务器的双向数据转发。  
  需要使用`context`机制，等待`ctx.Done()`避免`connect`函数立刻返回，连接关闭。
- 通过`switchOmega`插件，实现网络流量代理，[完整源码](https://github.com/KevinChens/youth-camp/tree/main/lesson1/proxy)：  
  ![img](https://s2.loli.net/2022/05/16/wSUBIHCJb9aTMDq.jpg)  

### 总结
本篇主要对Golang基础的重点易错点，几个常用标准库的使用方法进行了总结记录，并且通过两个小项目对Go语言上手实践，新手丝滑入门Golang。  
第一篇blog，大纲目录比较乱，总是修修改改，后边找个时间整理一下写blog的过程（挖坑）。欢迎大家评论转载，多多发表自己的意见或建议。  
（ps：blog写得真慢，要搞快点！！！）

### 引用参考
1. [Go by Example 中文版](https://gobyexample-cn.github.io)
2. [Golang标准库文档](https://studygolang.com/pkgdoc)

