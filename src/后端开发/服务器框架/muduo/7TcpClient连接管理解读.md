
# 1 Connector连接server
Connector负责根据IP地址创建socket连接server，并带有重试功能。其主要逻辑为：

1. 在connect函数中创建socket并尝试连接server，根据errno不同调用不同的函数
1. errno为EINPROGRESS，EINTR，EISCONN时，先进入connecting状态，初始化Channel对象，监听socket上的可读事件
1. 发送可读事件时，需要先是否Channel对象，**因为Connector类只负责建立连接并返回描述符，不负责监听。所有Channel可以reset，reset之后用于建立下一连接。**
1. 可写事件不表示连接建立，需要用getsockopt再确认一下err。之后通过newConnectionCallback_回调返回socket描述符给TcpClient


部分函数实现如下：
```cpp
void Connector::connecting(int sockfd)
{
    setState(kConnecting);
    assert(!channel_);
    channel_.reset(new Channel(loop_, sockfd)); //根据socket描述符初始化channel
    //绑定回调
    channel_->setWriteCallback(std::bind(&Connector::handleWrite, this));
    channel_->setErrorCallback(std::bind(&Connector::handleError, this));

    channel_->enableWriting(); //将可写事件添加到epoll wait list中
}

void Connector::handleWrite()
{
    if (state_ == kConnecting)
    {
        //本类只负责建立连接，确定连接后返回描述符，内部channel没用了，reset之后用于建立下一连接
        int sockfd = removeAndResetChannel();
        //有可写事件不表示连接建立，需要用getsockopt再确认一下err
        int err = sockets::getSocketError(sockfd);
        if (err)
        {
            retry(sockfd);
        }
        else if (sockets::isSelfConnect(sockfd)) //防止自连接
        {
            retry(sockfd);
        }
        else
        {
            setState(kConnected); //其他情况标识连接建立
            if (connect_)
            {
                newConnectionCallback_(sockfd); //调用回调，返回描述符
            }
            else
            {
                sockets::close(sockfd);
            }
        }
    }
    //...
}
```
Connector还有重试机制，连接不成功会启动EvenLoop定时任务重连：
```cpp
void Connector::retry(int sockfd)
{
    sockets::close(sockfd);
    setState(kDisconnected);
    if (connect_)
    {
        //设置定时任务，重试连接server
        loop_->runAfter(retryDelayMs_ / 1000.0,
                        std::bind(&Connector::startInLoop, shared_from_this()));
        retryDelayMs_ = std::min(retryDelayMs_ * 2, kMaxRetryDelayMs);
    }
    //...
}
```

# 2 TcpClient管理连接
TcpServer类通过**唯一的Connector**去连接server，连接建立后保存在类型为**TCP Connection**的成员变量connections_中。TcpServer是供App直接使用的，内嵌在用户程序中，创建和释放由用户程序控制。

1. 构造函数中设置connector的回调
```cpp
connector_->setNewConnectionCallback(std::bind(&TcpClient::newConnection, this, _1));
```

2. App层调用connect()函数，触发connector开始连接server
2. 连接建立后，connector回调newConnection，TcpClient创建TCP Connection对象（一对一关系）
```cpp
TcpConnectionPtr conn(new TcpConnection(loop_, //创建新的TcpConnection对象
                                        connName,
                                        sockfd,
                                        localAddr,
                                        peerAddr));

conn->setConnectionCallback(connectionCallback_);
conn->setMessageCallback(messageCallback_);
conn->setWriteCompleteCallback(writeCompleteCallback_);
conn->setCloseCallback(std::bind(&TcpClient::removeConnection, this, _1));
{
    MutexLockGuard lock(mutex_);
    connection_ = conn;
}
conn->connectEstablished(); //启动可读事件的监听
```
