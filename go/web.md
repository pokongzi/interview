## net/http
```go
func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello World!")
	})
	http.HandleFunc("/time/", func(w http.ResponseWriter, r *http.Request) {
		t := time.Now()
		timeStr := fmt.Sprintf("{\"time\": \"%s\"}", t)
		w.Write([]byte(timeStr))
	})

	http.ListenAndServe(":8080", nil)
}
```

### HandleFunc
此行代码会调用系统默认的ServeMux即DefaultServeMux，DefaultServeMux是http库定义的一个变量。
```go
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	es    []muxEntry // slice of entries sorted from longest to shortest.
	hosts bool       // whether any patterns contain hostnames
}
//最终执行方法的结构handler
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}

type muxEntry struct {
	h       Handler
	pattern string
}
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    DefaultServeMux.HandleFunc(pattern, handler)
    //DefaultServeMux 是一个*ServeMux，可以理解为一个服务。
}
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	if handler == nil {
		panic("http: nil handler")
	}
    mux.Handle(pattern, HandlerFunc(handler)) ///转成HandlerFunc类型 HandlerFunc属于handler类型
    //mux.Handle 在DefaultServeMux中注册路由和handler
    //之后DefaultServeMux的初始化完成
}

type HandlerFunc func(ResponseWriter, *Request) //参数固定
```

## http.ListenAndServe
http.ListenAndServe(":8080", nil)
*  监听端口
> ln, err := net.Listen("tcp", addr) //serve.go 2707
* 为请求创建一个连接
> c := srv.newConn(rw) 
* 开始服务
> go c.serve(ctx) 
* 初始化ServerHandler，并且调用他的ServeHTTP方法
> serverHandler{c.server}.ServeHTTP(w, w.req)  server方法里面
* ServeHttp方法会找出服务的一个ServeMux，如果没有用户自己没有初始化一个ServeMux，则会使用DefaultServeMux，也就是之前默认初始化的ServeMux，最后调用ServeMux的serveHTTP方法。
```go
 func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) { //serve.go 2686
   handler := sh.srv.Handler
   if handler == nil {
      handler = DefaultServeMux
   }
   if req.RequestURI == "*" && req.Method == "OPTIONS" {
      handler = globalOptionsHandler{}
   }
   handler.ServeHTTP(rw, req)
```
* 在ServeMux的serveHTTP方法中，找到处理函数并调用
```go
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {  //serve.go 2328
   if r.RequestURI == "*" {
      if r.ProtoAtLeast(1, 1) {
         w.Header().Set("Connection", "close")
      }
      w.WriteHeader(StatusBadRequest)
      return
   }
   h, _ := mux.Handler(r) //根据url在ServeMux中的m属性中找到处理函数，
   h.ServeHTTP(w, r)  
```

## 路由规则
* 如果都没有匹配到，返回404
* 最长匹配原则
* url末尾是‘/’，表示一个子树，后面可以跟其他路径；末位不是‘/’表示一个叶子，固定路径

## Handler第三方
github.com/julienschmidt/httprouter
可以支持restful格式的请求

## ListenAndServe
gracehttp.ListenAndServe(":"+*port, mux)
可以支持超时控制

http 应用程序重启时，如果直接 kill -9 使程序退出，然后在启动，会有以下几个问题：
旧的请求未处理完，如果服务端进程直接退出，会造成客户端链接中断（收到 RST）；
新请求打过来，服务还没重启完毕，造成 connection refused

信号机制；
子进程继承父进程的资源；
支持功能
平滑重启（Zero-Downtime restart server）；
平滑关闭；
多 Server 添加（包含 HTTP 、HTTPS）；
自定义日志组件；


## http Profile
简单，适合于持续性运⾏行行的应⽤用
在应⽤用程序中导⼊入 import _ "net/http/pprof"，并启动 http server 即可

### 查看方式1
http://<host>:<port>/debug/pprof/
go tool pprof http://<host>:<port>/debug/pprof/profile?seconds=10 (默认值为30秒
go-torch -seconds 10  http://127.0.0.1:8081/debug/pprof/profile //和go tool pprof 类似


## ⽂文件⽅方式输出 Profile
灵活性⾼高，适⽤用于特定代码段的分析
通过⼿手动调⽤用 runtime/pprof 的 API
go tool pprof [binary] [binary.prof]
go-torch -->火炬图

> go tool pprof prof cpu.prof
> cpu.prof	goroutine.prof	mem.prof 



查看时top 和list命令





