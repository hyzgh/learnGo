# ch3 打包和工具链

## 包

定义：一组功能清晰的、小的代码代码单元，可包含多个文件，这些文件需要放在同一个目录下。



## 导入

远程导入：从远程仓库导入包

命令导入：给导入的包重名命

空白标识符`_` ：可加在未使用的导入包前，避免编译错误

`init` 函数：导入包后进行初始化



## Go工具链

`go env` 查看Go环境信息

`go doc` 查看API文档

- `go doc <package>` 在终端直接查看帮助文档
- `go doc -http=:6060 &` 在后台运行web服务器，可通过浏览器访问

可通过该命令来给自己的代码生成文档，若要对包进行说明，则可在`doc.go`下写

`go get` 获取依赖包

# ch4 数组、切片和映射

补充点：

1. 数组和切片申明的小区别

    ```go
    // 创建长度为2的数组
    a := [...]int{1, 2}
    // 创建长度为2的切片
    b := []int{1, 2}
    ```

2. 第三个索引的使用

   `slice[i:j:k]` 表示长度为 j - i, 容量为 k - i

   可以令j等于k，新创建一个切片，然后再append，达到与原切片脱离开来的目的

   ```go
   source := []string{"Apple", "Orange", "Plum", "Banana", "Grape"}
   slice := source[2:3:3]
   slice = append(slice, "Kiwi")
   ```

3. 切片可以`append`到另一个切片

   ```go
   // 创建两个切片,并分别用两个整数进行初始化
   s1 := []int{1, 2}
   s2 := []int{3, 4}
   // 将两个切片追加在一起,并显示结果
   fmt.Printf("%v\n", append(s1, s2...))
   ```

4. 使用`range`遍历切片时，创建了每个元素的副本，而不是直接返回对该元素的引用

5. 多维切片中的切片长度可以是不一样的

   ```go
   // 创建一个整型切片的切片
   slice := [][]int{{10}, {100, 200}}
   // 为第一个切片追加值为 20 的元素
   slice[0] = append(slice[0], 20)
   ```

6. 在64位架构的机器上，一个切片需要24字节的内存，即指针，长度和容量。

# ch5 Go 语言的类型系统

补充点：

1. 编译器只允许为命名的用户定义的类型声明方法

2. 多态的写法

   ```go
   type notifier interface {
   	notify()
   }
   
   type user struct {
   	name  string
   	email string
   }
   
   func (u *user) notify() {
   	fmt.Printf("Sending user email to %s<%s>\n",
   		u.name,
   		u.email)
   }
   
   type admin struct {
   	name  string
   	email string
   }
   
   func (a *admin) notify() {
   	fmt.Printf("Sending admin email to %s<%s>\n",
   		a.name,
   		a.email)
   }
   
   func main() {
   	bill := user{"Bill", "bill@email.com"}
   	sendNotification(&bill)
   
   	lisa := admin{"Lisa", "lisa@email.com"}
   	sendNotification(&lisa)
   }
   
   // 对于实现了该接口的不同类型，可能会有不同的行为
   func sendNotification(n notifier) {
   	n.notify()
   }
   
   ```

3. `type embedding` 内部类型的方法可以直接被外部类型调用

   ```go
   type user struct {
   	name  string
   	email string
   }
   
   func (u *user) notify() {
   	fmt.Printf("Sending user email to %s<%s>\n",
   		u.name,
   		u.email)
   }
   
   type admin struct {
   	user  // Embedded Type
   	level string
   }
   
   func main() {
   	ad := admin{
   		user: user{
   			name:  "john smith",
   			email: "john@yahoo.com",
   		},
   		level: "super",
   	}
   
   	ad.user.notify()
   
   	// 直接调用内部类型的方法
   	ad.notify()
   }
   
   ```

4. 由于内部类型的提升，内部类型实现的接口会自动提升到外部类型。

   ```go
   type notifier interface {
   	notify()
   }
   
   type user struct {
   	name  string
   	email string
   }
   
   func (u *user) notify() {
   	fmt.Printf("Sending user email to %s<%s>\n",
   		u.name,
   		u.email)
   }
   
   type admin struct {
   	user
   	level string
   }
   
   func main() {
   	ad := admin{
   		user: user{
   			name:  "john smith",
   			email: "john@yahoo.com",
   		},
   		level: "super",
   	}
   
   	// 内部类型user实现了notifier接口，外部类型可以直接用
   	sendNotification(&ad)
   }
   
   func sendNotification(n notifier) {
   	n.notify()
   }
   
   ```

5. 假如外部类型也实现了接口，那么内部类型就不会被提升。

6. 当一个标识符的名字以小写字母开头时,这个标识符就是未公开的,即包外的代码不可见。如果一个标识符以大写字母开头,这个标识符就是公开的,即被包外的代码可见。

7. 利用内部类型的标识符提升到外部类型这条规则，我们可以这样写代码

   ```go
   // entities/entities.go
   package entities
   
   type user struct {
   	Name  string
   	Email string
   }
   
   type Admin struct {
   	user   // The embedded type is unexported.
   	Rights int
   }
   
   // main/main.go
   package main
   
   import (
   	"entities"
   	"fmt"
   )
   
   func main() {
   	a := entities.Admin{
   		Rights: 10,
   	}
   
   	// 虽然user对外不可见，但是由于内部类型提升，所以Name和Email可被直接访问
   	a.Name = "Bill"
   	a.Email = "bill@email.com"
   
   	fmt.Printf("User: %v\n", a)
   }
   ```

8.  嵌入类型提供了扩展类型的能力，而无需使用继承。

# ch6 并发

补充点：

1. 同步的方式：原子函数`atomic`，互斥锁`mutex`，通道`channel`
2. goroutine 在逻辑处理器上执行,而逻辑处理器具有独立的系统线程和运行队列。
3. 无缓冲的通道要求goroutine之间同时完成发送和接收。
4. 有缓冲的通道允许goroutine将数据放到缓冲中，当缓冲满时发送方会阻塞，当缓冲空时接收方会阻塞。
5. 对通道`close`意味着goroutine 依旧可以从通道接收数据，但是不能再向通道里发送数据。

# ch7 并发模式

学了几种并发模式，对Go的并发和通道的使用有了进一步的认识。

- 控制程序的生命周期
- 管理可复用的资源池
- 创建可以处理任务的goroutine池
