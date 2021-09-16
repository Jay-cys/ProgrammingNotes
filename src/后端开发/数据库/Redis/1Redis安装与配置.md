
# 1 Redis安装与启动
在官网下载Redis源码，解压后make install即可在/usr/local/bin下看到生成的可执行文件
```bash
tar xzf redis-6.0.9.tar.gz
cd redis-6.0.9
make
sudo make install
ls -l /usr/local/bin
-rwxr-xr-x 1 root root  6878864 Dec 20 01:27 redis-benchmark
-rwxr-xr-x 1 root root 11883600 Dec 20 01:27 redis-check-aof
-rwxr-xr-x 1 root root 11883600 Dec 20 01:27 redis-check-rdb
-rwxr-xr-x 1 root root  6892104 Dec 20 01:27 redis-cli
lrwxrwxrwx 1 root root       12 Dec 20 01:27 redis-sentinel -> redis-server
-rwxr-xr-x 1 root root 11883600 Dec 20 01:27 redis-server
```
生成的可执行文件的功能分别是

| 文件名 | 功能 |
| --- | --- |
| redis-server | Redis服务器 |
| redis-cli | 命令行客户端 |
| redis-benchmark | 性能测试工具 |
| redis-check-aof | AOF文件修复工具 |
| redis-check-rdb | RDB文件检查工具 |


# 2 启动和停止Redis
启动有三种方式

- `redis-server`：直接启动，默认使用6379端口
- `redis-server --port 6380`：指定自定义端口
- 使用初始化脚本，在源码目录utils下有名为**redis_init_script**的脚本范例，可以用于随系统自动启动


停止推荐如下方式，避免强制终止Redis导致数据同步到硬盘出现问题

- `redis-cli SHUTDOWN`


另外连接Redis也使用cli程序：`redis-cli -h localhost -p 6379`。

# 3 多数据库
Redis支持16个数据库，index从0开始（不支持设置数据库名），可以通过修改databse配置参数进行改变。客户端连接后，默认会选择0数据库。可以使用**`select index`**命令更换数据库。



