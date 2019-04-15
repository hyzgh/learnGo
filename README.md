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
