## 使用调试工具的场景
>1. 方便快速阅读复杂的工程
>2. 问题定位

## gdb
> gdb 是一个调试工具，可以调试很多种语言，当然其中就有golang。
## 准备工作
1. 由于gdb不能显示go的数据类型，所以在go的runtime源码下有一个python代码文件,runtime-gdb.py，提供给gdb一个扩展。程序启动之后可以通过 source命令，进行加载:source /usr/local/go/src/runtime/runtime-gdb.py
2. 编译： go build -gcflags "-N -l" （-N 禁止优化，-l禁止内联）
3. 带参数启动程序： gdb -arg  ./main(第一个参数是程序文件） -server=true（第二个参数）
4. attach:  gdb attach pid 
## 常用命令
1. 查看源码 list(l) file.go:line
2. 设置断点 break(b) client.go:40
3. 查看断点 info breakpoints
4. 删除断点 delete(d) num
5. 运行至断点 continue (c)
6. 打印变量 print(p) var
7. 运行 run(r)
8. 单步执行 next(n) 不进入函数
9. 单步执行 step(s) 进入函数
10. 退出函数 finish(f)
11. 查看goroutine info goroutines
## 实践
### (1)调试thrif客户端程序，查看发送rpc请求过程
1. go build -gcflags="-N -l" main.go client.go  handler.go  server.go
2. 启动服务端 nohup ./main -server=true &
3. 启动客户端 gdb -args ./main -server=false
4. 加载gdb扩展 source /usr/local/go/src/runtime/runtime-gdb.py
5. 设置断点 b client.go:40
6. 查看断点 info breakpoints
7. 运行至断点 r
8. 进入函数 s
9. next
10. clear 删除所有断点
### (2)调试thrif服务端程序,查看服务端接收到请求的处理过程
1. go build -gcflags="-N -l" main.go client.go  handler.go  server.go
2. 启动服务端 nohup ./main -server=true &
3. attach 到正在运行的服务 gdb attach 29709
4. 加载gdb扩展 source /usr/local/go/src/runtime/runtime-gdb.py
5. 设置断点 b handler.go:48（设置断点之后，客户端发送的请求都会被阻塞，如果客户端有超时设置，则会返回超时错误）
6. 查看断点 info breakpoints
7. 客户端发送一个请求 ./main -server=false
8. 运行至断点 c
9. 打印 p w.Op

## delve
> 由于gdb对golang支持比较差，只能支持一些简单的功能,比如断点，单步等，但对goroutine, 内置对象的打印等还是不怎么好。所以go有了自己的debug工具delve
###安装
> https://github.com/derekparker/delve
> go get -u github.com/go-delve/delve/cmd/dlv

### 准备工作
1. 编译
2. 带参数启动 dlv exec ./main -- 参数1 参数2
3. dlv attach pid

### 常用命令（跟gdb类似）
可以通过 help 命令查看

## 实践
### (1)调试thrif客户端程序，查看发送rpc请求过程
1. go build -gcflags="-N -l" main.go client.go  handler.go  server.go
2. 启动服务端 nohup ./main -server=true &
3. 启动客户端 dlv exec ./main -server=false
5. 设置断点 b gdb-dlv-thrift/client.go:40
6. 查看断点 bp
7. 运行至断点 c
8. 进入函数 s
9. 打印结构体 p w.Num1
10. clearallb 删除所有断点c
### (2)调试thrif服务端程序,查看服务端接收到请求的处理过程
1. go build -gcflags="-N -l" main.go client.go  handler.go  server.go
2. 启动服务端 nohup ./main -server=true &
3. attach 到正在运行的服务 dlv attach 40660
5. 设置断点 b handler.go:48（设置断点之后，客户端发送的请求都会被阻塞，如果客户端有超时设置，则会返回超时错误）
6. 查看断点 bp
7. 客户端发送一个请求 ./main -server=false
8. 运行至断点 c
9. 打印 p w.Op




