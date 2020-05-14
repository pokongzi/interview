# go问题总结


## 插件安装
> go get
> 有时候会安装失败，原因：golang.org官网被墙，需要从github上面下载 下好即可
> https://www.jianshu.com/p/4f2c476e189a


## gopath goroot gobin
GOROOT：Go 语言安装根目录的路径，也就是 GO 语言的安装路径。
GOPATH：若干工作区目录的路径。是我们自己定义的工作空间。
 你可以把 GOPATH 简单理解成 Go 语言的工作目录，它的值是一个目录的路径，也可以是多个目录路径，每个目录都代表 Go 语言的一个工作区（workspace）。我们需要利于这些工作区，去放置 Go 语言的源码文件（source file），以及安装（install）后的归档文件（archive file，也就是以“.a”为扩展名的文件）和可执行文件（executable file）
GOBIN：GO 程序生成的可执行文件（executable file）的路径。

## go源码文件
* 命令源码文件、
* 库源码文件
* 测试源码文件
![](./asset/2.png)



## 思考Go 语言在多个工作区中查找依赖包的时候是以怎样的顺序进行的？如果在多个工作区中都存在导入路径相同的代码包会产生冲突吗？
> 1，包的导入有几种路径方式，文稿中给出了通俗的方式，但是还有根据相对路径导入的方式，所以在导入包时，应该是先从当前项目(不知道可否叫当前工作区)开始查找，在根据GOPATH从前往后查找
> 2，不冲突，因为按顺序找到所需要的包就不往后找了。

### 结构
包名 package和目录不一定一样
为了不让该代码包的使用者产生困惑，我们总是应该让声明的包名与其父目录的名称一致
在同一个目录下的源码文件都需要被声明为属于同一个代码包.

引用
主程序 main()
命里行参数 :os.Args  os.Args[0]是方法路径 os.Args[1]才开始

## 构建明亮
go build
go build 用于测试编译包，主要检查是否会有编译错误，如果是一个可执行文件的源码（即是 main 包），就会直接生成一个可执行文件。
go install
go install 的作用有两步：第一步是编译导入的包文件，所有导入的包文件编译完才会编译主程序；第二步是将编译后生成的可执行文件放到 bin 目录下gobin目录，编译后的包文件放到 pkg 目录下
go get 
  命令go get会自动从一些主流公用代码仓库（比如 GitHub）下载目标代码包，并把它们安装到环境变量GOPATH包含的第 1 工作区的相应目录中。如果存在环境变量GOBIN，那么仅包含命令源码文件的代码包会被安装到GOBIN指向的那个目录。
-u：下载并安装代码包，不论工作区中是否已存在它们。
-d：只下载代码包，不安装代码包。
-t：同时下载测试所需的代码包。

go run 
编译并直接运行程序，它会产生一个临时文件,不会产生exe,直接在命令行输出程序执行结果，方便用户调试。

## go test 
在命名文件时需要让文件必须以_test结尾
每个测试用例函数需要以Test为前缀
func TestXXX( t *testing.T )

## iota
iota在const关键字出现时将被重置为0(const内部的第一行之前)，const中每新增一行常量声明将使iota计数一次.
iota只能在常量的表达式中使用。
```
const a = iota // a=0
const (
  b = iota     //b=0
  c            //c=1
)

const (
    IgEggs Allergen = 1 << iota         //  1 << 0 which is 00000001
    IgChocolate                         // 1 << 1 which is 00000010
    IgNuts                              // 1 << 2 which is 00000100
    IgStrawberries                      // 1 << 3 which is 00001000
    IgShellfish                         // 1 << 4 which is 00010000
)
```
## 赋值
类型推断
多个赋值

## Mutex 
https://www.jianshu.com/p/94bdaf3ad125
### 互斥锁 和读写锁
```go
func TestCounterWaitGroup(t *testing.T) {
	var mut sync.Mutex
	var wg sync.WaitGroup
	counter := 0
	for i := 0; i < 5000; i++ {
		wg.Add(1)
		go func() {
			defer func() {
				mut.Unlock()
			}()
			mut.Lock()
			counter++
			wg.Done()
		}()
	}
	wg.Wait()
	t.Logf("counter = %d", counter)
}
```

## channel的关闭广播
多个协程共用一个chanel如何关闭
1.最好使用close
2.接收第二个布尔形返回

## Context与子任务取消
根 Context: 通过context.Background()创建
子 Context :context.WithCancel(parentcontext)
ctx, cancel := context.WithCancel(context.Background())//cancel是方法，触发取消
当Context被取消，期子任务也取消
<-ctx.Done();取消消息

示例
``` go
func isCancelled(ctx context.Context) bool {
	select {
	case <-ctx.Done():
		return true
	default:
		return false
	}
}
func TestCancel(t *testing.T) {
	ctx, cancel := context.WithCancel(context.Background())
	for i := 0; i < 5; i++ {
		go func(i int, ctx context.Context) {
			for {
				if isCancelled(ctx) {
					break
				}
				time.Sleep(time.Millisecond * 5)
			}
			fmt.Println(i, "Cancelled")
		}(i, ctx)
	}
	cancel()
	time.Sleep(time.Second * 1)
}
```

### 仅需要任意任务完成
这样容易导致协程阻塞无法关闭影响性能  使用缓存通道避免
```
func runTask(id int) string {
	time.Sleep(10 * time.Millisecond)
	return fmt.Sprintf("The result is from %d", id)
}

func FirstResponse() string {
	numOfRunner := 10
	ch := make(chan string)
	for i := 0; i < numOfRunner; i++ {
		go func(i int) {
			ret := runTask(i)
			ch <- ret
		}(i)
	}
	return <-ch
}

func TestFirstResponse(t *testing.T) {
	t.Log("Before:", runtime.NumGoroutine())
	t.Log(FirstResponse())
	time.Sleep(time.Second * 1)
	t.Log("After:", runtime.NumGoroutine())

}
```

### 所有任务完成
1.sync.WaitGroup
2.for循环接收
```
func AllResponse() string {
	numOfRunner := 10
	ch := make(chan string, numOfRunner)
	for i := 0; i < numOfRunner; i++ {
		go func(i int) {
			ret := runTask(i)
			ch <- ret
		}(i)
	}
	finalRet := ""
	for j := 0; j < numOfRunner; j++ {
		finalRet += <-ch + "\n"
	}
	return finalRet
}
```

### 对象池 
通过缓存通道对容器进行限制
ch32

### sync.pool
为什么不能当作对象池
1.GC会清除sync.pool缓存对象
2.对象的有效期为下次GC之前，不适合做连接池，需要自己管理生命周期的池话
3.Golang的对象池严格意义上来说是一个临时的对象池，适用于储存一些会在goroutine间分享的临时对象.

https://www.cnblogs.com/sunsky303/p/9706210.html
sync.pool对象get
从proesser私有对象获取 协程安全.并从本地池冲删除该值。
不存在，到当前proesser共享池获取,并从共享队列中删除该值。
当前共享池是空到，到其他proesser获取.并删除共享池中的该值
如果都是空的，使用指定new函数产生一个新对象
put
如果放入的值为空，直接return.
检查当前goroutine的是否设置对象池私有值，如果没有则将x赋值给其私有成员，并将x设置为nil。
如果当前goroutine私有值已经被设置，那么将该值追加到共享列表。

### 测试

## fmt.Printf
String() Error()









