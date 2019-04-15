# Basic component

## packages

- 任何go程序都是由package组成，首个非空单词必须是`package`

## imports

单个import

```go
import "fmt"
```

多个import

```go
import (
	"fmt"
	"math"
)
```

## exported names

假如一个单词首字母是大写的，那么表示exported

## functions

形式：形参的标识符在前，类型在后；返回值放在最后面

```go
func add(x int, y int) int {
	return x + y
}
```

这样做的主要原因是为了提高易读性，特别是在涉及函数变量（函数指针）的时候

```go
f func(func(int,int) int, int) int
f func(func(int,int) int, int) func(int, int) int
```

同类型的形参可简写

```go
func add(x, y int) int {
	return x + y
}
```

返回值可有多个

```go
func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}
```

可给返回值命名，且return可简写，注意不要在长函数中简写，因为这样会降低可读性

```go
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}
```

## variables

申明格式

```go
# 单句
var c, python, java bool

# 块
var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)
```

有赋初值时可省略类型名

```go
var c, python, java = true, false, "no!"
```

当省略类型名时，编译器会自动推测，推测规则为：

- 右边是变量，则和变量的类型相同
- 右边是常量，则有可能是int, float64, complex128

```go
func main() {
	v := 42.12 + 0.5i// change me!
	fmt.Printf("v is of type %T\n", v)
}
```

在函数体内可简写

```go
func main() {
	k := 3
	fmt.Println(k)
}
```

基础类型

```
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128
```

申明的变量假如没有被显式赋初值，则会被赋`zero value`，即数值为0，布尔类型为false，字符串为""

申明常量，用`const`，不能用`:=`，因为`:=`是和`var`关联的，而`var`代表变量。

```go
func main() {
	const World = "世界"
	fmt.Println("Hello", World)
}
```

数值常量是高精度存储的，而且会根据上下文以合适的类型出现。

没有隐式类型转化，必须显式类型转化。

在函数体内，变量申明了但不使用会报错。

# Flow control statements

`for`，可以没有`()`，但一定要有`{}`

```go
for i := 0; i < 10; i++ {
	sum += i
}
```

go没有`while`，但可以用`for`代替

```go
sum := 1
for sum < 1000 {
	sum += sum
}
```

`if`，可以没有`()`，但一定要有`{}`

```go
if x < 0 {
	return sqrt(-x) + "i"
}
```

`if`可以像`for`那样先带个statement。假如申明了变量，则只能在`if`或后续的`else`中使用。

```go
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	} else {
		fmt.Printf("%g >= %g\n", v, lim)
	}
	// can't use v here, though
	return lim
}
```

`switch`, 满足其中一个`case`即执行其中的语句，不会再执行其他case的语句，可看成自带`break``。此外，case`不一定要接整数常量，它可以不是整数，可以不是常量。但要注意`case`后接的类型和`switch`比较的类型相同。

`switch`同样可以像`for`那样先带个statement。

```go
func main() {
	fmt.Print("Go runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
	}
}
```

`defer` 推迟执行，具有LIFO的性质

```go
func main() {
	defer fmt.Printf("1 ")
	defer fmt.Printf("2 ")
	fmt.Printf("3 ")
	// 将输出3 2 1
}
```

# More types

## Pointers

指针，和C不一样，Go没有指针的算术运算，即不支持`p = p + 10`这样的语句

## Structs

结构体，可将`(*p).X`写成`p.X`

```go
type Vertex struct {
	X int
	Y int
}

var (
	v1 = Vertex{1, 2}  // has type Vertex
	v2 = Vertex{X: 1}  // Y:0 is implicit
	v3 = Vertex{}      // X:0 and Y:0
	p  = &Vertex{1, 2} // has type *Vertex
)
```

## Arrays

数组

```go
func main() {
	var a [2]string
	a[0] = "Hello"
	a[1] = "World"
	fmt.Println(a[0], a[1])
	fmt.Println(a)

	primes := [6]int{2, 3, 5, 7, 11, 13}
	fmt.Println(primes)
}
```

## Slices

切片，本身不存储实际数据，类似于引用

```go
func main() {
	primes := [6]int{2, 3, 5, 7, 11, 13}

	var s []int = primes[1:4]
	fmt.Println(s)
}
```

切片数组，可以改变指向的范围

```go
func main() {
	r := []bool{true, true, true, false, false, false}
	t := []bool{true, true, true, false, false, false}
	fmt.Println(r)
	r = r[1:2]
	fmt.Println(r)
	r = r[0:3]
	fmt.Println(r)
    # 输出[true true false]，即最开始的[1,4]
	r = t
	fmt.Println(r)
}
```

切片，可省略下界或上界

`cap()` 查看容量，即从下界到数组最后一个元素的个数

`len()`查看长度，即从下界到上界的个数

切片为`nil`时，`cap`和`len`都为0


使用`make`来创建一维切片

```go
func main() {
	a := make([]int, 5)
	printSlice("a", a)

	b := make([]int, 0, 5)  // len(b)=0, cap(b)=5
	printSlice("b", b)

	c := b[:2]
	printSlice("c", c)

	d := c[2:5]
	printSlice("d", d)
}

func printSlice(s string, x []int) {
	fmt.Printf("%s len=%d cap=%d %v\n",
		s, len(x), cap(x), x)
}
```


使用`make`创建二维切片

```go
// 创建一个位数为[dx][dy]的切片
a := make([][]uint8, dx)
for i := range a {
	a[i] = make([]uint8, dy)
}
```

创建二维切片

```go
func main() {
	// Create a tic-tac-toe board.
	board := [][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	}

	// The players take turns.
	board[0][0] = "X"
	board[2][2] = "O"
	board[1][2] = "X"
	board[1][0] = "O"
	board[0][2] = "X"
	for i := 0; i < len(board); i++ {
		fmt.Printf("%s\n", strings.Join(board[i], " "))
	}
	/*
		X _ X
		O _ X
		_ _ O
	*/
}
```

可使用`append`函数向切片添加元素，假如切片容量不足，则容量会翻倍。

```go
func main() {
	var s []int
	s = append(s, 2, 3, 4)
	fmt.Println(s)
}
```

可使用`range`遍历切片，每次循环会有两个值，一个是元素的下标，一个是元素的值。可使用`_`忽略其中一个。

```go
func main() {
	pow := []int{4, 1, 5}
	for _, value := range pow {
		fmt.Printf("%d\n", value)
	}
    
    // 只有一个值只会得到下标
    for idx := range pow {
		fmt.Printf("%d\n", pow[idx])
	}
}
```
