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

# ch8 标准库

学了几个标准库：log, encoding/json, io

“应该花时间看一下标准库提供了什么，以及它是如何实现的——不仅要防止重新造轮子，还要理解Go语言的设计者的习惯，并将这些习惯应用到自己的包和API的设计上”。

# ch9 测试和性能

测试文件需要以`_test.go`结尾，用`go test`命令来测试。

基础单元测试例子

```go
// Sample test to show how to write a basic unit test.
package listing01

import (
	"net/http"
	"testing"
)

const checkMark = "\u2713"
const ballotX = "\u2717"

// TestDownload validates the http Get function can download content.
func TestDownload(t *testing.T) {
	url := "https://www.google.com"
	statusCode := 200

	t.Log("Given the need to test downloading content.")
	{
		t.Logf("\tWhen checking \"%s\" for status code \"%d\"",
			url, statusCode)
		{
			resp, err := http.Get(url)
			if err != nil {
				t.Fatal("\t\tShould be able to make the Get call.",
					ballotX, err)
			}
			t.Log("\t\tShould be able to make the Get call.",
				checkMark)

			defer resp.Body.Close()

			if resp.StatusCode == statusCode {
				t.Logf("\t\tShould receive a \"%d\" status. %v",
					statusCode, checkMark)
			} else {
				t.Errorf("\t\tShould receive a \"%d\" status. %v %v",
					statusCode, ballotX, resp.StatusCode)
			}
		}
	}
}

```



表组测试，顾名思义，进行多组数据的测试

例子

```go
// Sample test to show how to write a basic unit table test.
package listing08

import (
	"net/http"
	"testing"
)

const checkMark = "\u2713"
const ballotX = "\u2717"

// TestDownload validates the http Get function can download
// content and handles different status conditions properly.
func TestDownload(t *testing.T) {
	var urls = []struct {
		url        string
		statusCode int
	}{
		{
			"http://www.goinggo.net/feeds/posts/default?alt=rss",
			http.StatusOK,
		},
		{
			"http://rss.cnn.com/rss/cnn_topstbadurl.rss",
			http.StatusNotFound,
		},
	}

	t.Log("Given the need to test downloading different content.")
	{
		for _, u := range urls {
			t.Logf("\tWhen checking \"%s\" for status code \"%d\"",
				u.url, u.statusCode)
			{
				resp, err := http.Get(u.url)
				if err != nil {
					t.Fatal("\t\tShould be able to Get the url.",
						ballotX, err)
				}
				t.Log("\t\tShould be able to Get the url.",
					checkMark)

				defer resp.Body.Close()

				if resp.StatusCode == u.statusCode {
					t.Logf("\t\tShould have a \"%d\" status. %v",
						u.statusCode, checkMark)
				} else {
					t.Errorf("\t\tShould have a \"%d\" status. %v %v",
						u.statusCode, ballotX, resp.StatusCode)
				}
			}
		}
	}
}
```



模仿调用，用于模拟服务器，从而达到离线测试的目的

```go
// Sample test to show how to mock an HTTP GET call internally.
// Differs slightly from the book to show more.
package listing12

import (
	"encoding/xml"
	"fmt"
	"net/http"
	"net/http/httptest"
	"testing"
)

const checkMark = "\u2713"
const ballotX = "\u2717"

// feed is mocking the XML document we except to receive.
var feed = `<?xml version="1.0" encoding="UTF-8"?>
<rss>
<channel>
    <title>Going Go Programming</title>
    <description>Golang : https://github.com/goinggo</description>
    <link>http://www.goinggo.net/</link>
    <item>
        <pubDate>Sun, 15 Mar 2015 15:04:00 +0000</pubDate>
        <title>Object Oriented Programming Mechanics</title>
        <description>Go is an object oriented language.</description>
        <link>http://www.goinggo.net/2015/03/object-oriented</link>
    </item>
</channel>
</rss>`

// mockServer returns a pointer to a server to handle the get call.
func mockServer() *httptest.Server {
	f := func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(200)
		w.Header().Set("Content-Type", "application/xml")
		fmt.Fprintln(w, feed)
	}

	return httptest.NewServer(http.HandlerFunc(f))
}

// TestDownload validates the http Get function can download content
// and the content can be unmarshaled and clean.
func TestDownload(t *testing.T) {
	statusCode := http.StatusOK

	server := mockServer()
	defer server.Close()

	t.Log("Given the need to test downloading content.")
	{
		t.Logf("\tWhen checking \"%s\" for status code \"%d\"",
			server.URL, statusCode)
		{
			resp, err := http.Get(server.URL)
			if err != nil {
				t.Fatal("\t\tShould be able to make the Get call.",
					ballotX, err)
			}
			t.Log("\t\tShould be able to make the Get call.",
				checkMark)

			defer resp.Body.Close()

			if resp.StatusCode != statusCode {
				t.Fatalf("\t\tShould receive a \"%d\" status. %v %v",
					statusCode, ballotX, resp.StatusCode)
			}
			t.Logf("\t\tShould receive a \"%d\" status. %v",
				statusCode, checkMark)

			var d Document
			if err := xml.NewDecoder(resp.Body).Decode(&d); err != nil {
				t.Fatal("\t\tShould be able to unmarshal the response.",
					ballotX, err)
			}
			t.Log("\t\tShould be able to unmarshal the response.",
				checkMark)

			if len(d.Channel.Items) == 1 {
				t.Log("\t\tShould have \"1\" item in the feed.",
					checkMark)
			} else {
				t.Error("\t\tShould have \"1\" item in the feed.",
					ballotX, len(d.Channel.Items))
			}
		}
	}
}

// Item defines the fields associated with the item tag in
// the buoy RSS document.
type Item struct {
	XMLName     xml.Name `xml:"item"`
	Title       string   `xml:"title"`
	Description string   `xml:"description"`
	Link        string   `xml:"link"`
}

// Channel defines the fields associated with the channel tag in
// the buoy RSS document.
type Channel struct {
	XMLName     xml.Name `xml:"channel"`
	Title       string   `xml:"title"`
	Description string   `xml:"description"`
	Link        string   `xml:"link"`
	PubDate     string   `xml:"pubDate"`
	Items       []Item   `xml:"item"`
}

// Document defines the fields associated with the buoy RSS document.
type Document struct {
	XMLName xml.Name `xml:"rss"`
	Channel Channel  `xml:"channel"`
	URI     string
}

```

可以根据规则写一些示例，然后在包的文档里显示。

基准测试。
