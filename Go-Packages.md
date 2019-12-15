# builtin

描述了内置的基本数据类型、常量、函数。

需要记忆的点：

- `string`类型是一成不变的
- `complex64`的实数部分和虚数部分是`float32`；`complex128`的实数部分和虚数部分是`float64`
- `int`, `uint`至少为32bit
- `uintptr`是个整数类型，可存储任意指针
- `byte`是`uint8`的alias
- `rune`是`int32`的alias
- `iota`是常量0，可用于优雅地申明数列
- `nil`是pointer, channel, func, interface, map, slice type的零值

```go
// 用于往slice添加元素
// 假如切片容量足够容纳新增元素，则reslice
// 否则，会allocate一块新的内存区域
func append(slice []Type, elems ...Type) []Type
// 教科书用法
slice = append(slice, elem1, elem2)
slice = append(slice, anotherSlice...)

// 用于copy slice的元素
// 若dst和src重叠，则src会覆盖dst
// 返回值为复制的元素个数，等于min(len(src), len(dst))，注意不是cap
func copy(dst, src []Type) int

// 用于删除map中的key
// 若m为nil或key不存在，则不操作，不会panic
func delete(m map[Type]Type1, key Type)

// 用于得到某些数据结构的长度
// 包括array, pointer to array, slice, map, string, channel
// 若传入nil，则返回0，不会panic
func len(v Type) int

// 用于得到某些数据结构的容量
// 包括array, pointer to array, slice, channel
// 若传入nil, 则返回0，不会panic
func cap(v Type) int

// 用于slice, map, chan，且只适用于这三种类型
// 用于为其分配内存并初始化，并返回一个非指针的对象
// map可传入size，但可能会被忽略
// channel传入size，表示buffer的size
func make(t Type, size ...IntegerType) Type

// 用于任意类型
// 用于为其分配内存并初始化，并返回一个指向对象指针
func new(Type) *Type

// 用于构造一个复数
func complex(r, i FloatType) ComplexType
// 返回复数的实数部分
func real(c ComplexType) FloatType
// 返回复数的虚数部分
func imag(c ComplexType) FloatType

// 用于关闭channel，要求channel必须是发送方
// 获取channel中的内容：
// x, ok := <- c
// 若channel中的元素未被取完，则ok为true
// 若已被取完，则ok为false，且x为零值
func close(c chan<- Type)

// 用于终止程序
// panic后会立即结束当前函数的执行，接着执行defer语句
// 会递归着panic上层的函数，直到顶层，除非被recover了
// 入参用于传递panic的信息
// panic(nil)也是会panic，但不建议这么写，因为nil表示没有错误，有歧义
func panic(v interface{})

// 用于恢复程序
// panic后可用recover来恢复执行
// 多次recover会得到nil
// 需要写在defer中，且不能直接defer recover()
// 教科书写法：
// defer func() {
//     recover()
// }
// panic()
func recover() interface{}


// 用于调试，输出信息到stderr
// 该函数不保证在后续版本一定存在
func print(args ...Type)
func println(args ...Type)

// 用于表示错误
type error interface {
	Error() string
}
```



# bytes

bytes库主要包括两个类和五类函数。

两个类是指`Reader`和`Buffer`。

五类函数是指`Comparison`, `Inspection`, `Prefix/suffix`, `Replacement`, `Splitting & joining`。

```go
Reader，实现的接口有io.Reader, io.ReaderAt, io.WriterTo, io.Seeker, io.ByteScanner, and io.RuneScanner，用于读[]byte
1. 创建Reader
func NewReader(b []byte) *Reader

2. 读取r的长度
func (r *Reader) Len() int

3. 将r中的数据读到b中
func (r *Reader) Read(b []byte) (n int, err error)

4. 将r中从off开始的数据读到b中
func (r *Reader) ReadAt(b []byte, off int64) (n int, err error)

5. 读r中的一个byte
func (r *Reader) ReadByte() (byte, error)

6. 读r中的一个rune
func (r *Reader) ReadRune() (ch rune, size int, err error)

7. 将r设置为从b读
func (r *Reader) Reset(b []byte)

8. 设置r的读取位置，whene的值可以是io.SeekStart, io.SeekEnd
func (r *Reader) Seek(offset int64, whence int) (int64, error)
```





# compress/gzip

```go
// 用gzip压缩及解压内容
var buf bytes.Buffer
zw := gzip.NewWriter(&buf)

// Setting the Header fields is optional.
zw.Name = "a-new-hope.txt"
zw.Comment = "an epic space opera by George Lucas"
zw.ModTime = time.Date(1977, time.May, 25, 0, 0, 0, 0, time.UTC)

_, err := zw.Write([]byte("A long time ago in a galaxy far, far away..."))
if err != nil {
	log.Fatal(err)
}

if err := zw.Close(); err != nil {
	log.Fatal(err)
}

zr, err := gzip.NewReader(&buf)
if err != nil {
	log.Fatal(err)
}

fmt.Printf("Name: %s\nComment: %s\nModTime: %s\n\n", zr.Name, zr.Comment, zr.ModTime.UTC())

if _, err := io.Copy(os.Stdout, zr); err != nil {
	log.Fatal(err)
}

if err := zr.Close(); err != nil {
	log.Fatal(err)
}
```



# context

```go
/* 
context可用于跨API且跨进程的通信，产生背景是服务器接受到一条请求后，会起多个协程来处理这个请求，这些协程之间需要共享某些信息，比如用户标识符、授权token、请求deadline。用context，既可以方便地共享这些数据，还可以统一取消所有相关的协程，方便管理资源。
context可携带deadline, cancellation signals, values等
函数的调用链可通过context串起来，所以要求这个context是有亲缘关系的，即用WithCancel, WithDeadline, WithTimeout, or WithValue这些方法来创建子context。当一个context被cancel时，所有的子context也会被cancel。
WithCancel, WithDeadline, and WithTimeout这三个方法创建子context时会返回一个CancelFunc，调用这个函数可以取消子context以及它的所有孩子。假如没有调用CancelFunc，会导致该context泄漏，直到被父context取消。可以用go vet工具检测Context的泄漏情况。
同一个context可传递给不同的goroutine，它是并发安全的。

规范：
1. 不要将context嵌套在其他结构体里
2. 要作为函数入参的第一个，经典申明是func DoSomething(ctx context.Context, arg Arg) error 
3. 不要传递nil context，假如不确定要传什么，可传context.TODO
4. 使用context Values来存储跨API、跨进程的请求数据，不要存储一些函数的可选参数
*/


// Context定义
type Context interface {
    // Done returns a channel that is closed when this Context is canceled
    // or times out.
    Done() <-chan struct{}

    // Err indicates why this context was canceled, after the Done channel
    // is closed.
    Err() error

    // Deadline returns the time when this Context will be canceled, if any.
    Deadline() (deadline time.Time, ok bool)

    // Value returns the value associated with key or nil if none.
    Value(key interface{}) interface{}
}

/*
Done() 可以接收信号，知道什么时候应该取消。
Err() 可以在Done()接到信号后，查看原因。
Deadline() 可以查看deadline，评估是否应该执行。设置Deadline处理IO操作比较常用。
Value() 可以查看键值对
*/


// Context派生
// Background returns an empty Context. It is never canceled, has no deadline,
// and has no values. Background is typically used in main, init, and tests,
// and as the top-level Context for incoming requests.
func Background() Context

// WithCancel returns a copy of parent whose Done channel is closed as soon as
// parent.Done is closed or cancel is called.
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)

// A CancelFunc cancels a Context.
type CancelFunc func()

// WithTimeout returns a copy of parent whose Done channel is closed as soon as
// parent.Done is closed, cancel is called, or timeout elapses. The new
// Context's Deadline is the sooner of now+timeout and the parent's deadline, if
// any. If the timer is still running, the cancel function releases its
// resources.
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)

// WithValue returns a copy of parent whose Value method returns val for key.
// 官方建议key的类型自己来创建，因为这样做可以避免冲突
func WithValue(parent Context, key interface{}, val interface{}) Context


// 避免key冲突的方法，将key的类型对外隐藏
// The key type is unexported to prevent collisions with context keys defined in
// other packages.
type key int

// userIPkey is the context key for the user IP address.  Its value of zero is
// arbitrary.  If this package defined other context keys, they would have
// different integer values.
const userIPKey key = 0

func NewContext(ctx context.Context, userIP net.IP) context.Context {
    return context.WithValue(ctx, userIPKey, userIP)
}

func FromContext(ctx context.Context) (net.IP, bool) {
    // ctx.Value returns nil if ctx has no value for the key;
    // the net.IP type assertion returns ok=false for nil.
    userIP, ok := ctx.Value(userIPKey).(net.IP)
    return userIP, ok
}


// 使用timeout的方法

```





# flag

参考官方文档。



# fmt

`Printf`

| verb   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| %v     | 输出值                                                       |
| %+v    | 输出键-值                                                    |
| %#v    | 输出包名、类型名、键-值                                      |
| %T     | 类型                                                         |
| %%     | 百分号                                                       |
| %t     | 布尔值                                                       |
| %b     | 二进制的值                                                   |
| %c     | Unicode编码的字符                                            |
| %d     | 十进制的值                                                   |
| %o     | 八进制的值                                                   |
| %x或%X | 十六进制的值                                                 |
| %U     | 十六进制表示的Unicode值                                      |
| %s     | 字符串                                                       |
| %p     | 地址                                                         |
| %f     | 浮点数，默认精度是小数点后6位                                |
| %e     | 浮点数，科学计数法，默认精度是小数点后6位                    |
| %g     | 浮点数，有效数字，尽可能地输出所有位数                       |
| +      | 添加正负号                                                   |
| -      | 设置宽度时默认在左边补全空格，该符号可设置在右边补全空格     |
| 0      | 用0代替空格进行补全                                          |
| #      | 对于八进制，十六进制等，加上提示符，如八进制为0，十六进制为0x |

`%f` 可指定浮点数的宽度和精度

```
%f     default width, default precision
%9f    width 9, default precision
%.2f   default width, precision 2
%9.2f  width 9, precision 2
%9.f   width 9, precision 0
```

`%g` 可指定浮点数的有效数字位数，对于12.345，`%.3g` 将输出 12.3

若要能对某个自定义类型输出，只要对它定义`String() string`方法即可：

```go
func (t *T) String() string {
    return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
}
// 只有当t是*T类型时，才会调用上面那个函数
// 若要当t是T类型和*T类型都都调用上面那个函数，需要将上面的*T改成T
fmt.Printf("%v\n", t)
```







# image

```go
// 查看图片的分辨率, 类型
config, format, err := image.DecodeConfig(reader)
if err != nil {
	log.Fatal(err)
}
fmt.Println("Width:", config.Width, "Height:", config.Height, "Format:", format)


// 读取图片的内容
```



# io

```go

```



# io/ioutil

```go
1. 创建一个临时目录
func TempDir(dir, prefix string) (name string, err error)

2. 创建一个临时文件
func TempFile(dir, pattern string) (f *os.File, err error)

3. 读取文件所有内容
func ReadAll(r io.Reader) ([]byte, error)
```



# log

会将内容输出到stderr，且会增加一些信息（如日期时间）。



# net/http

```go
// 发送请求
func (c *Client) Do(req *Request) (*Response, error)

// 给请求添加context
func (r *Request) WithContext(ctx context.Context) *Request
```






# json

注意只有当结构体内的成员是公开时，才能在Marshal的时候被识别，成为json文件的一部分。

进行Unmarshal时，假如json中有的字段而结构体没有，则这个字段会被忽略，不影响解析。也就是说，可进行json文件的部分解析。同理，假如Marshal时，结构体中的字段不想转到JSON文件中，可以将其tag设置为"-"。

解析时，结构体的某个字段的匹配优先级为tag -> 导出名精确匹配 -> 导出名模糊匹配。

omitempty表示当字段为零值时忽略它，而tag为"-"表示直接忽略它。

UnmarshalText函数和UnmarshalJSON函数的区别是什么？



```go
// 当不知道json文件的格式时，可这样解析
b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)
var f interface{}
err := json.Unmarshal(b, &f)
if err == nil {
  m := f.(map[string]interface{})
  for k, v := range m {
    switch vv := v.(type) {
      case string:
      fmt.Println(k, "is string", vv)
      case float64:
      fmt.Println(k, "is float64", vv)
      case []interface{}:
      fmt.Println(k, "is an array:")
      for i, u := range vv {
        fmt.Println(i, u)
      }
      default:
      fmt.Println(k, "is of a type I don't know how to handle")
    }
  }
}


// 若将结构体的成员设置为指针类型，则若json不存在相应的字段，则为nil
type IncomingMessage struct {
  Cmd *Command
  Msg *Message
}

// 对于流的Encoders和Decoders
de := json.NewDecoder(os.Stdin)
enc := json.NewEncoder(os.Stdout)
for {
  var v map[string]interface{}
  if err := dec.Decode(&v); err != nil {
    log.Println(err)
    return
  }
  for k := range v {
    if k != "Name" {
      delete(v, k)
    }
  }
  if err := enc.Encode(&v); err != nil {
    log.Println(err)
  }
}


// 格式化输出，带缩进
type Road struct {
	Name   string
	Number int
}
roads := []Road{
	{"Diamond Fork", 29},
	{"Sheep Creek", 51},
}

b, err := json.Marshal(roads)
if err != nil {
	log.Fatal(err)
}

var out bytes.Buffer
json.Indent(&out, b, "=", "\t")
out.WriteTo(os.Stdout)


```



# path/filepath

和路径操作相关的包。

```go
1. 获取绝对路径
func Abs(path string) (string, error)

absPath, _ := filepath.Abs("main.go")
fmt.Println(absPath)
// 在/Users/hyz/code/go-lang/src/tmp/main/下运行输出/Users/hyz/code/go-lang/src/tmp/main/main.go


2. 获取路径的最后一个元素
func Base(path string) string


3. 简化路径，可理解为模拟路径操作
func Clean(path string) string

path := filepath.Clean("./dir/../main.go")
fmt.Println(path)
// 输出main.go


4. 输出某个路径所在的目录
func Dir(path string) string

fmt.Println(filepath.Dir("/foo/bar/baz.js"))
// 输出/foo/bar


5. 解析链接路径
func EvalSymlinks(path string) (string, error)

path, _ := filepath.EvalSymlinks("/etc/resolv.conf")
fmt.Println(path)
// 在macOS下输出/private/var/run/resolv.conf


6. 取文件拓展名
func Ext(path string) string

fmt.Printf("No dots: %q\n", filepath.Ext("index"))
fmt.Printf("One dot: %q\n", filepath.Ext("index.js"))
fmt.Printf("Two dots: %q\n", filepath.Ext("main.test.js"))
/* 输出：
No dots: ""
One dot: ".js"
Two dots: ".js"
*/


7. 判断是不是绝对路径
func IsAbs(path string) bool

8. 将多个元素连接起来，生成一个路径
func Join(elem ...string) string


```



# pprof

```go
var cpuprofile = flag.String("cpu", "cpu", "write cpu profile to file")
var memprofile = flag.String("mem", "mem", "write memory profile to this file")


func main() {
	//runtime.MemProfileRate = 1
	flag.Parse()
	if *cpuprofile != "" {
		f, err := os.Create(*cpuprofile)
		if err != nil {
			log.Fatal(err)
		}
		pprof.StartCPUProfile(f)
		defer pprof.StopCPUProfile()
	}
	// start here

	// end here
	if *memprofile != "" {
		f, err := os.Create(*memprofile)
		if err != nil {
			log.Fatal(err)
		}
		pprof.WriteHeapProfile(f)
		f.Close()
		return
	}
}
```



# strings

```go
1. 在母串中从后往前找子串
func LastIndex(s, substr string) int
// 源码使用了Rabin-Karp算法，其思想利用了哈希函数

2. 在母串中从后往前找某个字符
func LastIndexByte(s string, c byte) int

3. 
```



# sync

```go
// 用于只执行一次某个函数
type Once
func (o *Once) Do(f func())

// 互斥锁
type Mutex
func (m *Mutex) Lock()
func (m *Mutex) Unlock()


```



# sync/atomic

```go
// 底层硬件支持的原子操作
类别：
Add
CompareAndSwap
Load
Store
Swap
```



# testing

`testing.T` 普通测试

`testing.B` 性能测试

`testing.M` 可以传参



# time

```bash

```

