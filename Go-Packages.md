# bufio

`bufio`包是对基本I/O包的包装，实现了带缓存的I/O，并对文本的扫描提供了一些帮助。

主要实现了三个类，Reader, Writer, Scanner。

```go
// 创建一个带buffer的Reader，并指定buffer的size
func NewReaderSize(rd io.Reader, size int) *Reader

// 创建一个带buffer的Reader，buffer大小为默认值4K
func NewReader(rd io.Reader) *Reader

// 获取buffer的size
func (b *Reader) Size() int

// 更改Reader的底层Reader
func (b *Reader) Reset(r io.Reader)

// 读取n个byte，且不会移动Reader的读取位置
// 注意在下次Read后，返回的字节数组不应再次被访问
// 假如读取的byte少于n，则err不为nil
// 假如buffered的byte少于n，则err为io.EOF
// 假如n大于buffer的size，则err为bufio.ErrBufferFull
func (b *Reader) Peek(n int) ([]byte, error)

// 跳过n个byte
// 假如跳过的byte少于n，则err不为nil
// 假如buffered的byte少于n，则err为io.EOF
func (b *Reader) Discard(n int) (discarded int, err error)

// 读取byte到p中，读取的个数取决于buffer的字节数以及p的len
// 返回的n表示读取的字节数
// 若发现数据已经读完，则err会返回EOF
func (b *Reader) Read(p []byte) (n int, err error)

// 读取一个byte
func (b *Reader) ReadByte() (byte, error)

// 撤销读取一个byte的操作
// 要求上次操作是ReadByte()
func (b *Reader) UnreadByte() error

// 读取一个rune
// 假如是非法rune，则会读取一个byte，并返回unicode.ReplacementChar
func (b *Reader) ReadRune() (r rune, size int, err error)

// 撤销读取一个rune的操作
// 要求上一次操作是ReadRune()
func (b *Reader) UnreadRune() error

// 返回当前可以从buffer中读取的字节数
func (b *Reader) Buffered() int

// 读取字节，直到delim字符，返回包括delim字符的字符数组
// 假如不包含delim字符，会报错
// 在下次Read后，返回的line会失效，若要不失效，用ReadBytes和ReadString
func (b *Reader) ReadSlice(delim byte) (line []byte, err error)

// 读取一行
// 假如返回的isPrefix为true，表示还没读完一整行
// 在下次Read后，会失效
func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error)

// 读取字节，直到delim字符，返回包括delim字符的字符数组
// 假如不包含delim字符，会报错
func (b *Reader) ReadBytes(delim byte) ([]byte, error)

// 是对ReadBytes()的包装，返回string
func (b *Reader) ReadString(delim byte) (string, error)

// 实现了io.WriteTo接口
// 可能会读多次底层Reader
// 若底层Reader已实现io.WriteTo，则会直接调用而不做buffer
func (b *Reader) WriteTo(w io.Writer) (n int64, err error)

// 创建一个带buffer的writer
func NewWriterSize(w io.Writer, size int) *Writer

// 创建一个带buffer的writer，且buffer大小为默认值
func NewWriter(w io.Writer) *Writer

// 返回buffer的大小
func (b *Writer) Size() int

// 更改底层的writer，会忽略已buffer的内容及error
func (b *Writer) Reset(w io.Writer)

// 将buffer的中的内容输出到底层writer中
func (b *Writer) Flush() error

// 返回buffer的剩余空间
func (b *Writer) Available() int

// 返回已buffer的内容大小
func (b *Writer) Buffered() int

// 写内容到buffer中
func (b *Writer) Write(p []byte) (nn int, err error)

// 写一个byte到buffer中
func (b *Writer) WriteByte(c byte) error

// 写一个Rune到buffer中
func (b *Writer) WriteRune(r rune) (size int, err error)

// 写一个string到buffer中
func (b *Writer) WriteString(s string) (int, error)

// 从Reader读取内容到Writer中
func (b *Writer) ReadFrom(r io.Reader) (n int64, err error)

// 创建一个带buffer的Reader与Writer
// 注意入参是*bufio.Reader及*bufio.Writer
func NewReadWriter(r *Reader, w *Writer) *ReadWriter

// 创建一个scanner
func NewScanner(r io.Reader) *Scanner

// 返回非EOF的error
func (s *Scanner) Err() error

// 返回最近一次扫描得到的token，字符数组
func (s *Scanner) Bytes() []byte

// 返回最近一次扫描得到的token，字符串
func (s *Scanner) Text() string

// 扫描下一个token
// 假如扫描终止，会得到false，可通过Err()查看错误
func (s *Scanner) Scan() bool

// 自定义buffer的大小
// 入参为初始的buffer大小以及在scan过程可重新分配的buffer最大大小
// 不可在scan后调用该方法
func (s *Scanner) Buffer(buf []byte, max int)

// 设置SplitFunc，用以分隔token
func (s *Scanner) Split(split SplitFunc)

// SplitFunc，扫描获取单个byte
func ScanBytes(data []byte, atEOF bool) (advance int, token []byte, err error)

// SplitFunc，扫描获取单个rune
func ScanRunes(data []byte, atEOF bool) (advance int, token []byte, err error)

// SplitFunc，扫描获取单个行
func ScanLines(data []byte, atEOF bool) (advance int, token []byte, err error)

// SplitFunc，扫描获取单个单词，空格的定义为unicode.IsSpace()
func ScanWords(data []byte, atEOF bool) (advance int, token []byte, err error)
```



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
// 言外意义，假如dst的空间不够，copy不会帮忙增长空间
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



# container/heap

定义了heap的interface，实现这些接口即可实现一个堆。

提供了一些对heap的操作，如Push, Pop, Init, Fix, Remove等。

```go
// heap的定义
type Interface interface {
    sort.Interface
    Push(x interface{}) // add x as element Len()
    Pop() interface{}   // remove and return element Len() - 1
}

// 用于主动触发调整堆结构，O(logn)
func Fix(h Interface, i int)

// 用于建堆, O(n)
func Init(h Interface)

// 用于取出最小元素，O(logn)
func Pop(h Interface) interface{}

// 用于插入元素，O(logn)
func Push(h Interface, x interface{})

// 删除第i个元素并返回，O(logn)
func Remove(h Interface, i int) interface{}

// 例子
h.Push(1)  // O(1)
h.Push(2)  // O(1)
h.Push(3)  // O(1)
h.Push(4)  // O(1)
Init(h)  // O(n)
Push(h, 3)  // O(logn)
Pop(h)  // O(logn)
h[0] = 10  // O(1)
Fix(h，0)  // O(logn)
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





# cropto

## sha256

```go
package main

import (
	"crypto/sha256"
	"fmt"
)

func main() {
	h := sha256.New()
	h.Write([]byte("hello world\n"))
	fmt.Printf("%x", h.Sum(nil))
}
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



## image/jpeg



## image/png



# io

```go
// 从src拷贝数据到dst
// 返回拷贝的字节数和遇到的第一个错误，若成功，返回的err为nil而不是EOF
func Copy(dst Writer, src Reader) (written int64, err error)

// 用于建立获取一个Reader，在从r读取的时候，同时往w读取，两者同时进行，没有buffer
func TeeReader(r Reader, w Writer) Reader


```



## io/ioutil

```go
// 创建一个临时目录
func TempDir(dir, prefix string) (name string, err error)

// 创建一个临时文件
func TempFile(dir, pattern string) (f *os.File, err error)

// 读取文件所有内容
func ReadAll(r io.Reader) ([]byte, error)
```



# log

会将内容输出到stderr，且会增加一些信息（如日期时间）。

```go
// 输出行号以及文件名
log.SetFlags(log.LstdFlags | log.Lshortfile)
```



# net

```go
// 用于建立连接，网络类型可以是TCP、UDP、IP等
func Dial(network, address string) (Conn, error)

// 用于得到address，可作为Dial的入参
func JoinHostPort(host, port string) string

// 用于将address分解得到host, post
func SplitHostPort(hostport string) (host, port string, err error)

// 得到Listener，该接口有三个方法，分别是Accept, Close, Addr
func Listen(network, address string) (Listener, error)

// 监听连接，等待下一个连接
Accept() (Conn, error)

// 关闭连接，任何阻塞的Accept操作会解锁并返回错误
Close() error

// 得到listener的网络地址
Addr() Addr

```



## net/http

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



**unmarshal**

**unmarshal**

```go
// 用于解析json格式的数据，存储到Go的数据结构中
// 假如v为空或者不是指针，返回InvalidUnmarshalError
func Unmarshal(data []byte, v interface{}) error

```



假如Unmarshal时不知道json的内容，因为json顶层是一个object，所以可以Unmarshal到一个map中。

若json的数据数据结构转为interface{}时，有以下类型转化关系：

| json     | go                     |
| -------- | ---------------------- |
| booleans | bool                   |
| numbers  | float64                |
| strings  | string                 |
| arrays   | []interface{}          |
| objcets  | map[string]interface{} |
| null     | nil                    |

unmarshal到map[string]interface{}的具体用法：

```go
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
```



假如已经知道json的内容，可将其Unmarshal到一个struct中。

json object转化为go struct时，字段需要找到目的地，有以下寻址规则：优先寻找json tag的字段，其次寻找完全匹配的字段，再其次忽略大小写寻找匹配的字段。如果寻址不到，则该字段被忽略（假如不想忽略，可参考Decoder.DisallowUnknownFields）。

找到目的地后，不同的数据结构有不同的Unmarshal规则，内置类型的规则如下：

| json的数据结构 | Go的数据结构                     | 规则                                                         |
| -------------- | -------------------------------- | ------------------------------------------------------------ |
| booleans       | bool                             |                                                              |
| numbers        | numeric types(excluding complex) | 当溢出时，会继续Unmarshal，返回UnmarshalTypeError描述出现的第一个错误 |
| string         | string                           | 当解析quoted string时，非法UTF-8或者UTF-16并不会报错，而是把它们置为U+FFFD |
| array          | slice                            | 将slice的长度设置为0，然后一个个添加到slice中。              |
| array          | array                            | 若go array的长度比json array短，则会截断。若go array的长度比json array长，则会置零值。 |
| object         | map                              | 若map为nil，则会自动new。若map不为nil，则保留已有的键值对，然后将object中的键值对存储到其中。要求map的键为int, string或实现了encoding.TextUnmarshaler。 |
| nil            | null                             |                                                              |
|                | pointer                          | 若json为null，则go为nil。否则，说明有内容可以解析，若go中指针不为nil，则直接解析，否则会自动new再解析。 |
|                |                                  |                                                              |



可自定义Unmarshal规则，具体如下：

如果实现了Unmarshaler，则会调用它的UnmarshalJson方法，即使json的值为null。

如果实现了encoding.TextUnmarshaler，且json的值为quoted string，则会调用UnmarshalText方法，传入一个unquoted string。



# path

用于处理正斜杆拆分的路径，不适用于反斜杠拆分的路径。假如要处理反斜杠拆分的路径，使用path/filepath。

```go
// 返回路径的最后一个元素
func Base(path string) string

// 返回最短的等价路径，实现用了词法分析
func Clean(path string) string

// 返回路径目录，即除了最后一个元素外的路径
func Dir(path string) string

// 返回文件拓展名
func Ext(path string) string

// 判断是否是绝对路径
func IsAbs(path string) bool

// 连接路径，并最简
func Join(elem ...string) string

// 判断文件名是否与通配符匹配
func Match(pattern, name string) (matched bool, err error)

// 将path分成dir+file
func Split(path string) (dir, file string)
```

源码批注：

- Match函数的实现占据了path包过半的代码，它具体是怎么实现的？
- Clean是怎么实现的？





## path/filepath

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

9. 遍历目录
func Walk(root string, walkFn WalkFunc) error
type WalkFunc func(path string, info os.FileInfo, err error) error

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



# reflect

## rerlect三条定律

参考：https://blog.golang.org/laws-of-reflection

反射用于在运行阶段获取某些值的数据结构，是元编程的一部分。许多编程语言的反射模型是不同的，有的甚至不支持。因此，这里的反射指的是Go的反射模型，或许不能通用到其他语言。

有两个核心概念，types和interfaces。

Go是静态语言，在编译时，每个值都有固定的type。另外，对于不同类型的值，即使底层类型一致，也都需要显式类型转化才可以。比较特殊的一种type是interface，只要某个值实现了这个interface定义的方法集合，就可以存储它。虽然interface可以存储不同type的值，但这不违反静态语言的特性，因为申明为interface的值的类型是interface，这个值的类型是固定的。空接口可以存储所有的值。

反射和interface密切相关，必须理解好type和interface这两个概念。

一个interface的变量，会存储两个东西，(value, concrete type)，而不是(value, interface type)。注意将interface变量复制给interface时，它存储的仍是(value, concrete type)，而不存储这个interface的相关信息。

给一个interface赋值的时候，分两种情况，一种是具体变量，另一种是interface变量。对于具体变量，会检查这个变量是否实现了这个interface，因此不需要断言。对于interface变量，会检查interface type，假如有父集关系，不需要断言，假如没有就需要断言。因此，不管是具体变量还是interface变量，都可以直接赋值给空接口。例子：

```go
// os.OpenFile返回的是struct，属于普通变量，可以直接复制给io.Reader，不需要断言
// io.Reader赋值给io.Writer，在赋值时候右边的类型是io.Reader，没有父集关系，需要断言
var r io.Reader
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
if err != nil {
    return nil, err
}
r = tty
var w io.Writer
w = r.(io.Writer)

// io.ReadWriter复制给io.Writer，在赋值时候右边的类型是io.ReadWriter，有父集关系，不需要断言
var r io.ReadWriter
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
if err != nil {
    return nil, err
}
r = tty
var w io.Writer
w = r.(io.Writer)
```

第一条定律。从interface变成反射对象的反射。反射对象指Type和Value，反射对象的具体方法参考package。反射对象有很多方法。第一，为了简化API，getter和setter只设置了最大的数据类型，比如对于所有带符号数字，会使用int64。第二，对于自定义类型，Kind方法会返回它的底层类型。比如type MyInt int会返回int，而不是MyInt。

第二条定律。从反射对象到interface的反射。通过`func (v Value) Interface() interface{}`可以封包，得到原来的interface。封包是解包的逆过程。**对于fmt.Println等函数，封包解包都能输出正确的值，但这是Println做的手脚，有待研究它动了什么手脚。**

第三条定律。修改一个反射对象的前提，是它的值可以被设置。是否可以设置值是Value的一个属性，假如对一个不可以设置值的Value调用Set方法，会panic。假如反射对象是由指针转化而来的，那么是可以修改的，否则它是一个拷贝，通过set方法不会修改到原来的值，我们认为是没有意义的，因此不允许修改。



## reflect package

有两个主要类型，Type和Value。Type可以通过reflect.TypeOf(x)得到，Value可以通过reflect.ValueOf(x)得到。另外，Value有通过Type()方法可以得到Type。

```go
// Value
// 解析interface{}，得到Value
func ValueOf(i interface{}) Value

// 得到Value，表示的是一个指向零值的指针
func New(typ Type) Value

// 假如Value存的是指针，可以通过Elem方法得到指针指向的变量，这样可以修改这个变量
func (v Value) Elem() Value

// 得到Kind，可以是Bool, Int, Float32, Ptr等
func (v Value) Kind() Kind

// 得到Type反射对象
func (v Value) Type() Type

// 得到interface{}
func (v Value) Interface() (i interface{})

// 是否可以设置值
func (v Value) CanSet() bool

// 得到struct的字段数，假如不是struct会panic
func (v Value) NumField() int

// 得到Value，struct的第i个字段，假如不是struct或越界会panic
func (v Value) Field(i int) Value


// Type
// 解析interface{}，得到Type
func TypeOf(i interface{}) Type

// Type的方法，可以得到struct第i个字段的详细信息，具体看StructField
Field(i int) StructField
```



## 常用场景

### 读取并修改struct内的字段

```go
func main() {
	t := T{23, "skidoo"}
	s := reflect.ValueOf(&t).Elem()
	typeOfT := s.Type()

	// read
	for i := 0; i < s.NumField(); i++ {
		f := s.Field(i)
		fmt.Printf("%d: %s %s = %v\n", i,
			typeOfT.Field(i).Name, f.Type(), f.Interface())
	}

	// write
	s.Field(0).SetInt(77)
	s.Field(1).SetString("Sunset Strip")
	fmt.Println("t is now", t)
}
```





# sort

```go
// 给slice排序，自定义规则
sort.Slice(nodes, func(i, j int) bool)

// 给[]int排序
func Ints(a []int)
```



# strconv

用于字符串的转化，这个包含有一堆函数，这些函数可以分成以下几类：

- 将bool, float, int, uint 转化为string，函数前缀为Format或Append
- 将string转化为bool, float, int, uint，函数前缀为Parse
- 将string转化为string，函数前缀为Quote
- 其他函数，为了方便使用而做的封装，如Atoi, Itoa



这里有一个quote的概念需要理解，有以下几种情况：

- string转string(quote)：反转义控制字符、被IsPrint定义的不可打印字符
- string转string(ASCII)：反转义控制字符、被IsPrint定义的不可打印字符、非ASCII字符
- string转string(graphic)：反转义控制字符、被IsGraphic定义的不可打印字符、非ASCII字符



简短说明：

```go
// 往dst添加bool对应的字符数组
func AppendBool(dst []byte, b bool) []byte

// 往dst添加float对应的字符数组, prec指定小数位数, bitSize指定float的位度
func AppendFloat(dst []byte, f float64, fmt byte, prec, bitSize int) []byte

// 往dst添加string对应的字符数组，且前后有双引号"
func AppendQuote(dst []byte, s string) []byte

// 往dst添加一个rune对应的字符数组，且前后有单引号'
func AppendQuoteRune(dst []byte, r rune) []byte

// 往dst添加一个rune对应的string(ASCII)
func AppendQuoteRuneToASCII(dst []byte, r rune) []byte

// 往dst添加一个rune对应的string(grahphic)
func AppendQuoteRuneToGraphic(dst []byte, r rune) []byte

// 往dst添加string对应的string(ASCII)
func AppendQuoteToASCII(dst []byte, s string) []byte

// 往dst添加string对应的string(graphic)
func AppendQuoteToGraphic(dst []byte, s string) []byte

// 往dst添加unit对应的字符数组，base表示进制
func AppendUint(dst []byte, i uint64, base int) []byte

// string转int，等价于ParseInt(s, 10, 0)
func Atoi(s string) (int, error)

// 判断string能否Unquote，规则见源码
func CanBackquote(s string) bool

// bool转string
func FormatBool(b bool) string

// float转string，fmt表示形式，prec表示小数位数，bitSize表示float位数
func FormatFloat(f float64, fmt byte, prec, bitSize int) string

// int转string，base表示进制
func FormatInt(i int64, base int) string

// uint转string，base表示进制
func FormatUint(i uint64, base int) string

// 判断是否是Graghpic字符
func IsGraphic(r rune) bool

// 判断是否是可打印字符
func IsPrint(r rune) bool

// int转string，等价于FormatInt(int64(i), 10)
func Itoa(i int) string

// string转bool
func ParseBool(str string) (bool, error)

// string转float
func ParseFloat(s string, bitSize int) (float64, error)

// string转int
func ParseInt(s string, base int, bitSize int) (i int64, err error)

// string转uint
func ParseUint(s string, base int, bitSize int) (uint64, error)

// string转string(quote)，所谓quote，是指加个双引号且对引号内的控制字符及不可打印字符进行转移
func Quote(s string) string

// rune转string(quote)
func QuoteRune(r rune) string

// rune转string(ASCII)，即被定义为IsPrint的非ASCII字符需要进行反转义，比如'☺'转成'\u263a'
func QuoteRuneToASCII(r rune) string

// rune转string(graphic)，即被定义为Grahpic的非ASCII字符需要进行反转义
func QuoteRuneToGraphic(r rune) string

// string(quote)转string(ASCII)
func QuoteToASCII(s string) string

// string(quote)转string(ASCII)
func QuoteToGraphic(s string) string

// string(quote)转string，进行转义，比如'\u263a'转成"☺"
func Unquote(s string) (string, error)

// 解析出第一个quote，一般是双引号"
func UnquoteChar(s string, quote byte) (value rune, multibyte bool, tail string, err error)
```



源码批注：

- IsPrint和IsGraphic是本地实现的，功能和unicode的是一致的。之所以不用unicode包，是因为这样不依赖于unicode并因此不需要拉取所有的unicode tables，减少了不必要的链接。
- `const intSize = 32 << (^uint(0) >> 63)` 获取`int`的位数，32位或者64位

- 良好的代码风格：

	```go
	// &&的优先级高于||，当有&&与||混用的情况时，分开多行写可读性会更好
	if intSize == 32 && (0 < sLen && sLen < 10) ||
		intSize == 64 && (0 < sLen && sLen < 19) {
	    doSomething();
	}
	```

	

- 





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



