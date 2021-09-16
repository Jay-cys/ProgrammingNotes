RPC是远程过程调用的简称，是分布式系统中不同节点间流行的通信方式。互联网时代，RPC已经和IPC一样成为一个不可或缺的基础构件。
Go语言的标准库也提供了一个简单的RPC实现，包名为**net/rpc**。下面是使用该包的基本框架：

- 在Server端需要完成两件事
   - 明确服务名称和接口方法：方法只能有两个可序列化的参数，第二个参数是指针类型，并返回error类型，同时必须是公开的方法
   - 注册RPC服务并绑定到某一端口进行监听
```go
//构造一个类型，添加Hello方法
type HelloService struct{}
//注意Go语言的RPC规则
func (p *HelloService) Hello(request string, reply *string) error {
	*reply = "hello:" + request
	return nil
}
func main() {
	rpc.RegisterName("HelloService", new(HelloService)) //注册结构体类型为RPC服务
	//这里建立TCP连接用于RPC服务，也支持HTTP协议的RPC
	listener, err := net.Listen("tcp", ":1234")
	if err != nil {
		log.Fatal("Listen TCP error:", err)
	}

	conn, err := listener.Accept()
	if err != nil {
		log.Fatal("Accept error:", err)
	}

	rpc.ServeConn(conn)
}
```


- 在client需要完成一件事：访问端口连接并调用接口方法
```go
func main() {
	//尝试与server端建立TCP连接
	client, err := rpc.Dial("tcp", "localhost:1234")
	if err != nil {
		log.Fatal("Dial RPC error:", err)
	}

	var reply string
	//调用远程方法并传入参数
	err = client.Call("HelloService.Hello", "barret", &reply)
	if err != nil {
		log.Fatal("call RPC function error:", err)
	}

	fmt.Println(reply)
}
```

**需要注意的是：标准库的RPC默认采用Go语言特有的gob编码，因此从其它语言调用Go语言实现的RPC服务将比较困难**。虽然可以通过额外的工作支持跨语言，但是其实没必要，我们可以使用ProtoBuf和gRPC等框架支持跨语言。
