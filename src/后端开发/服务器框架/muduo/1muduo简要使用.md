
# 1 muduo简介

## 1.1 编译安装
```shell
git clone https://github.com/chenshuo/muduo.git
cd muduo
sudo apt install libboost-dev # 安装boost库
# 修改build.sh中的INSTALL_DIR，默认是在~/build目录下，我们可以改为/usr，直接安装到系统目录
sudo ./build.sh install
```
之后就可以看到/usr/include/muduo中相关的头文件，/usr/lib中有相应的静态链接文件。使用如下命令可以进行正常编译：
```makefile
# 以编译muduo的echo例子为例，指定静态库和include路径
g++ main.cc echo.cc -lmuduo_net -lmuduo_base -lpthread -I/home/barret/muduo -o echo
```

## 1.2 目录结构

- base：基础库，多是多线程相关的类
- net和net/poller：基于reactor模式的网络库，核心是EventLoop，用于处理IO事件和定时器
- net附属模块
   - http：简易的http服务器
   - inspect：基于http的窥探器，用于报告进程状态
   - protorpc：模仿protobuf RPC的实现
- example：示例代码

# 2 使用示例
可以查看附带的Finger了解如何一步步实现一个简单的echo server：代码地址：examples/twisted/finger

## 2.1 simple目录示例
`examples/simple`目录下实现了几个简单的TCP网络服务程序，功能列举如下：

- discard：丢弃所有收到的数据
- daytime：服务端accept连接之后，以字符串形式发送当前时间，然后主动断开连接
- time：服务端accept连接之后，以二进制形式发送当前时间（从Epoch到现在的秒数），然后主动断开连接；我们需要一个客户程序来把收到的时间转换为字符串
- echo：回显服务，把收到的数据发回客户端。
- chargen：服务端accept连接之后，不停地发送测试数据。




## 2.2 文件传输示例
`examples/filetransfer/`三种实现方式：

1. 一次读取全文件内容，一次发送
1. 每次读取64K内容，多次发送
1. 使用智能指针管理文件描述符，自动释放

# 3 最大并发连接数问题与解决
当文件描述符用尽时，会存在这样的问题：本进程的文件描述符已经达到上限，无法为新连接创建socket文件描述符。但是，既然没有socket文件描述符来表示这个连接，我们就无法close(2)它。程序继续运行，再一次调用epoll_wait。这时候epoll_wait会立刻返回，因为新连接还等待处理，listening fd还是可读的。这样**程序立刻就陷入了busy loop，CPU占用率接近100％**。这既影响同一event loop上的连接，也影响同一机器上的其他服务。

所以设置一个最大并发连接数的限制是必须的。有两个基本的方案：

1. 准备一个**空闲的文件描述符**。遇到这种情况，先关闭这个空闲文件，获得一个文件描述符的名额；再accept(2)拿到新socket连接的描述符；随后立刻close(2)它，这样就优雅地断开了客户端连接；最后重新打开一个空闲文件，把“坑”占住，以备再次出现这种情况时使用。**muduo的acceptor类内部实现了这种方法**。**这种方法在多线程下有竞争问题，多个线程可能同时需要空闲文件描述符**
1. 可以自己**设一个稍低一点的soft limit**，如果超过soft limit就主动关闭新连接，这样就可避免触及“文件描述符耗尽”这种边界条件。比方说当前进程的max file descriptor是1024，那么我们可以在连接数达到1000的时候进入“拒绝新连接”状态，这样就可留给我们足够的腾挪空间。


在muduo中可以很简单的在App层实现方案2（尽管库里已经实现方法1），为Server类添加一个最大限制的成员变量，当新连接到来时，判断是否超过最大限制：
```cpp
class XxxServer
{
	//...
    atomic_int numConnected_; //当前活动连接数，原子类型
  	const int kMaxConnections_;//最大连接限制
};
void XxxServer::onConnection(const TcpConnectionPtr &conn)
{
    if (conn->connected())
    {
        ++numConnected_;
        if (numConnected_ > kMaxConnections_) //判断是否超过最大的并发连接数限制，
        {                                     //超过，立刻关闭新连接
            conn->shutdown();
            conn->forceCloseWithDelay(3.0); // > round trip of the whole Internet.
        }
    }
    else
    {
        --numConnected_;
    }
}
```
