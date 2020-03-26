### why
使用调试工具的场景：
1. 方便快速阅读复杂的工程
2. 问题定位

### gdb
> gdb 是一个调试工具，可以调试很多种语言，当然其中就有golang。
#### 准备工作
1. 由于gdb不能显示go的数据类型，所以在go的runtime源码下有一个python代码文件,runtime-gdb.py，提供给gdb一个扩展。程序启动之后可以通过 source命令，进行加载:source /usr/local/go/src/runtime/runtime-gdb.py
2. 编译： go build -gcflags "-N -l" （-N 禁止优化，-l禁止内联）
3. 带参数启动程序： gdb -arg  ./main(第一个参数是程序文件） -server=true（第二个参数）
4. attach:  gdb attach pid 
#### 常用命令
1. 查看源码 list(l) file.go:line
2. 设置短点 break(b) client.go:40
3. 查看短点 info breakpoints
4. 删除短点 delete(d) num
5. 打印变量 print(p) var
6. 运行 run(r)
7. 单步执行 next(n) 不进入函数
8. 单步执行 step(s) 进入函数
9. 退出函数 finish(f)
10. 查看goroutine info goroutines
### 实践
#### 调试thrif客户端程序，查看rpc请求过程
1. go build -gcflags="-N -l" main.go client.go  handler.go  server.go
2. 启动服务端 ./main -server=true
3. 启动客户端 gdb -args ./main -server=false
4. 加载gdb扩展 source /usr/local/go/src/runtime/runtime-gdb.py
5. 设置断点 b client.go:40
6. 查看断点 info breakpoints



