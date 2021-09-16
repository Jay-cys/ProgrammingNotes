
# 1 定时器函数
muduo EventLoop有三个定时器函数：
```cpp
class EventLoop : noncopyable
{
    /// Runs callback at 'time'.
    /// Safe to call from other threads.
    ///
    TimerId runAt(Timestamp time, TimerCallback cb);
    ///
    /// Runs callback after @c delay seconds.
    /// Safe to call from other threads.
    ///
    TimerId runAfter(double delay, TimerCallback cb);
    ///
    /// Runs callback every @c interval seconds.
    /// Safe to call from other threads.
    ///
    TimerId runEvery(double interval, TimerCallback cb);
    ///
    /// Cancels the timer.
    /// Safe to call from other threads.
    ///
    void cancel(TimerId timerId);
};
```
**回调函数在EventLoop对象所属的线程发生**，与onMessage()、onConnection()等网络事件函数在同一个线程。muduo的TimerQueue采用了**平衡二叉树**来管理未到期的timers，因此这些操作的事件复杂度是O(logN) 。

# 2 核心代码解读
TimerQueue采用**timerfd_create创建一个timer的描述符，通过epoll复用函数和其他socket描述符一样被监听**。当timer超时时，会触发epoll监听到**可读事件**，TimerQueue就可以在epoll返回后触发回调函数，执行超时之后的任务（TimerQueue内部有个timer list，可以在超时后执行多个任务，因为可能多个timer同时超时）。需要注意的时，增删timer是在EventLoop IO监听线程，防止出现多线程同时修改的同步问题。

## 2.1 创建timer描述符并监听
```cpp
TimerQueue::TimerQueue(EventLoop *loop)
    : loop_(loop),
      timerfd_(createTimerfd()),       //创建timer描述符
      timerfdChannel_(loop, timerfd_), //根据timer描述符创建channel
      timers_(),
      callingExpiredTimers_(false)
{
    //设置channel的可读回调，即timer超时回调函数
    timerfdChannel_.setReadCallback(std::bind(&TimerQueue::handleRead, this));
    // we are always reading the timerfd, we disarm it with timerfd_settime.
    timerfdChannel_.enableReading(); //加入epoll wiat list
}

void TimerQueue::handleRead()
{
    loop_->assertInLoopThread();
    Timestamp now(Timestamp::now());
    readTimerfd(timerfd_, now); //timer描述符的可读事件，需要read一下

    std::vector<Entry> expired = getExpired(now); //查找timer list并删除超时的timer

    callingExpiredTimers_ = true;
    cancelingTimers_.clear();
    // safe to callback outside critical section
    for (const Entry &it : expired)
    {
        it.second->run(); //调用timer对象的超时回调函数
    }
    callingExpiredTimers_ = false;
    reset(expired, now);
}
```

## 2.2 增删timer
以下两个函数是在IO线程执行，加入到EventLoop的pending factors中，等待loop循环中执行，避免了线程同步问题。
```cpp
void TimerQueue::addTimerInLoop(Timer *timer)
{
    loop_->assertInLoopThread();
    bool earliestChanged = insert(timer);

    if (earliestChanged)
    {
        resetTimerfd(timerfd_, timer->expiration()); //设置timer描述符的超时时间
    }
}

void TimerQueue::cancelInLoop(TimerId timerId)
{
    loop_->assertInLoopThread();
    assert(timers_.size() == activeTimers_.size());
    ActiveTimer timer(timerId.timer_, timerId.sequence_);
    ActiveTimerSet::iterator it = activeTimers_.find(timer); //在timer list中查找要取消的timer
    if (it != activeTimers_.end())
    { //删除此timer，这样timer描述符可读事件来了，会被丢掉不处理
        size_t n = timers_.erase(Entry(it->first->expiration(), it->first));
        assert(n == 1);
        (void)n;
        delete it->first; // FIXME: no delete please，删除timer对象
        activeTimers_.erase(it);
    }
    else if (callingExpiredTimers_)
    {
        cancelingTimers_.insert(timer);
    }
    assert(timers_.size() == activeTimers_.size());
}
```
