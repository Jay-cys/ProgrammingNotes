Channel类是IO事件回调的分发器（dispatcher），它在handleEvent()中根据事件的具体类型分别回调ReadCallback、WriteCallback等。**每个Channel对象服务于一个文件描述符，但并不拥有fd**，在析构函数中也不会close(fd)。Channel也使用muduo一贯的boost::function来表示函数回调，它不是基类。这样用户代码不必继承Channel，也无须override虚函数。Channel与EventLoop的内部交互有两个函数`EventLoop::updateChannel(Channel*)`和`EventLoop::removeChannel(Channel*)`。客户需要在Channel析构前自己调用`Channel::remove()`。

相关代码结构图示如下：![muduo.png](.assets/1607418236994-006695ed-5855-4d26-9d2e-b0d96bf5831d.png)
