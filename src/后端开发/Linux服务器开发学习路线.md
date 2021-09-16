> 引用自：[https://studygolang.com/articles/31307#reply0](https://studygolang.com/articles/31307#reply0)


# Linux入门到精通篇

## 一、Linux开发环境

- [x] 了解Linux环境搭建，了解LinuxC编程
- [x] 了解Linux安装，命令使用，shell编程




## 二、Linux环境编程

- [x] Linux C编程：文件操作、文件指针
- [x] 并发方案，包括：互斥锁、自旋锁、原子操作
- [x] 实现线程池，包括：线程队列，任务队列，条件变量
- [x] CPU与进程的关系，包括：进程操作，进程与CPU粘合，进程通信
- [x] 数据库操作，包括：数据库封装，sql语句封装，网络连接封装




## 三、网络编程

- [ ] DNS请求器，包括：UDP通信，DNS协议，协议解析
- [ ] 实现http请求器 TCP客户端，包括：TCP编程，HTTP请求协议
- [ ] 百万级并发服务器 TCP服务器，包括：tcp，网络io，Linux系统



> 总结：把以上知识点内容掌握之后你的Linux就已经比较成熟了，达到了一个Linux开发工程师的水平了。


---


# Linux后台开发篇

## 一、算法于设计

1. 排序与查找，包括：插入排序、快速排序、希尔排序、桶排序、基数排序、归并排序
1. 常用算法，包括：布隆过滤器、字符串匹配KMP算法、回溯算法、贪心算法、推荐算法、深度 广度优先
1. 常用的数据结构，包括：平衡二叉树、红黑树、B-树、KMP算法、栈/队列
1. 常用设计模式，包括：单列模式、责任链模式、过滤器模式、发布订阅模式、代理模式、工厂模式




## 二、后台组件编程

1. 持久化 MySQL，包括：MySQL安装配置与远程连接、数据操作源于SQL语句、存储过程与事务处理、SQL函数，运算，临时表、防数据丢失 备份与恢复、MySQL建库建表建索引
1. 消息队列 ZeroMQ，包括：ZMQ编译安装与开发环境搭建、publisher-subscriber模式实现、request-response模式实现、Router-Dealer模式实现、消息队列—性能分析
1. 缓存 Redis，包括： Redis编译安装配置、客户端全局唯一ID保存机制、Redis消息队列机制 发布订阅、Redis事务实战、Redis安全性能，数据备份与恢复、Redis分布式锁详解
1. 反向代理 Nginx，包括： Nginx开发介绍、反向代理负载均衡配置详解、自定义协议upstream开发、子域名映射、服务器后台攻击预防、nginx双虚拟主机
1. Restful Http，包括：Http第三方接口实现、异步Http请求、ngrok与Restlet、长连接与短链接
1. 协调服务 ZooKeeper，包括：ZK编译安装与C API开发环境、集群管理与服务注册、节点创建与监控、分布式锁的实现、ZK伪集群部署与服务管理
1. NoSQL MongoDB，包括：MongDB安装与开发介绍、MongoDB备份与恢复、MongoDB文档操作、全文检索与正则表达式、MongoDB建库建集合




## 三、代码工程化

1. 架构工程，包括：工程参数配置与编译 cmake、代码规范与命名规则、文件命名与变量命名规则、脚本配置工具 autoconf、代码工程组织架构 Makefile
1. 管理代码，包括： 分布式版本控制系统 git、远程仓库，标签管理、 github与码云、创建仓库，导入，checkout、svn环境搭建与原理、 分支管理 冲突解决、产品代码版本管理 SVN




## 四、网络服务

1. 源码实现，包括：服务器IO核心— epoll编程实战、客户端多网络连接机制poll、文件IO管理select
1. 框架，包括：高性能的时间循环 libev、跨平台异步I/O libuv、跨平台的C++库Boost.Asio、事件通知库libevent
1. 理论，包括：阻塞型 BIO、异步IO AIO、非阻塞型IO NIO




## 五、开源框架

1. TCP协议栈，包括：基于DPDK的高性能用户态协议栈 f-stack、基于Netmap单线程协议栈 NtyTcp、精简版tcp协议栈 LWIP
1. 并发性，包括：用OpenCL的C++ GPU计算库 Boost.Compute、Intel线程构件块 Intel TBB、并行编程的异构系统的开放标准 OpenCL、C11的反应性编程库 C React
1. 数据库，包括：Redis数据库的C客户端库 hiredis、Facebook的嵌入键值的快速存储 RocksDB、用于Sqlite3的C++对象关系映射 hiberlite
1. 国际化，包括：Unicode 和全球化支持的C、C++ 和Java库 IBM ICU、不同字符编码之间的编码转换库 libiconv、GNU gettext
1. 压缩，包括：非常紧凑的数据流压缩库 Zlib、快速压缩和解压缩 Snappy、非常快速的压缩算法 LZ4、单一的C源文件，紧缩/膨胀压缩库 Miniz
1. 日志，包括：设计非常模块化，并且具有扩展性 Boost.Log、灵活添加日志到文件，系统日志 Log4cpp、添加日志到你的C应用程序 templog、C日志库，只包含单一的头文件 easyloggingpp
1. 多媒体库，包括：开源音频库—跨平台的音频API OpenAL、网络实时流媒体通信 WebRTC、音频和音乐数字信号处理库 Maximilian、C++易用和高效的音频合成 Tonic
1. 序列化，包括：快速数据交换格式和RPC系统 Cap’n Proto、协议缓冲，谷歌的数据交换格式 ProtoBuf、高效的跨语言IPC/RPC Thrift、内存高效的序列化库 FlatBuffers
1. XML库，包括：Gnome的xml C解析器和工具包 LibXml2、单快速的CCML解析器 TinyXML2、简单快速的XML解析器 PugiXML、C的xml解析器 LibXml++
1. 脚本，包括：小型快速脚本引擎 Lua、谷歌的快速JavaScript引擎 V8、嵌入式脚本语言 ChaiScript、
1. Json库，包括：进行编解码和处理Jason数据的C语言库 Jansson、C语言中的JSON解析和打印库 ibjson、轻量级的JSON库 libjson、C/C++的Jason解析生成器 Frozen
1. 数学库，包括：高质量的C线性代数库 Armadillo、数学图形模板库 GMTL、用于个高精度计算的C/C库 GMP、高级C++模板头文件库 Eigen
1. 安全，包括：SSL，TLS和DTLS协议的安全通信库 GnuTLS、功能齐全的，开源加密库 Openssl、有关加密方案的免费的C库 Cryto
1. Web应用框架，包括：安全快速开源Web服务器 Lighttpd、于Qt库的web框架 QDjango、高性能的HTTP和反向代理web服务器 Nginx
1. 网络库，包括：C异步网络开发库 Dyad.c、多协议文件传输库 Curl、高速模块化的异步通信库 ZeroMQ、C++面向对象网络工具包 ACE
1. 异步事件，包括：事件通知库 libevent、 跨平台异步I/O libuv、功能齐全，高性能的时间循环 libev、网络和底层I/O编程的跨平台的C++库 Boost.Asio
1. 协程，包括：纯c版的协程框架 ntyco、C++11实现协程库, golang风格 libgo、微信支持8亿用户同时在线的底层IO库 libco




## 六、性能测试

1. 调试库，包括：Boost测试库 Boost.Test、内存调试性能分析工具 Valgrind、谷歌C++测试框架 GoogleTest、内存分配跟踪库 MemTrack
1. 测试库，包括：单元测试框架 minUnit、测试用例编写 libtap、轻量级的C单元测试框架 UnitTest、自动化测试用例 gtest和luatest
1. 性能工具，包括：高性能代码构建系统 tundra、Http压测工具 WRK、 网站压测工具 webbench、高性能构建系统 FASTBuild



> 总结：以上知识点比较多、但是想要真正了解后台开发就必需要了解跟熟悉的掌握这些知识点内容。在你以后的工作中看的是会要用到。熟练掌握以上知识点内容你的水平就达到了后台开发工程师了。

