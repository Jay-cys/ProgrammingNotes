Unix域套接字主要用于**本机进程间的通信**，传输速度是TCP的两倍。


# 1 Unix域地址结构
```c
#define UNIX_PATH_MAX 108

struct sockaddr_un{
    sa_family_t sun_family; //AF_UNIX或AF_LOCAL
    char sun_path[UNIX_PATH_MAX]; //pathname, unix域用路径名表示地址
};
```



# 2 Unix域套接字注意

- sun_path最好使用**绝对路径**
- unix域协议支持流式和数据报式套接口
- unix域协议connect发现监听队列满里，会立刻返回**ECONNREFUSED**， 和TCP不同。




# 3 代码示例

## server
```c
#include <unistd.h>
#include <sys/un.h>
#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>

#define ERR_EXIT(m)\
    do\
    {\
        perror(m);\
        exit(EXIT_FAILURE);\
    }while(0)

void handle_subprocess(int conn)
{
    char buf[1024];
    int ret;
    while(1)
    {
        memset(buf, 0, sizeof(buf));
        ret = read(conn, buf, sizeof(buf));
        if (ret == -1)
        {
            if (ret == EINTR)
                continue;
            ERR_EXIT("read failed");
        }
        if (ret == 0)
        {
            printf("client closed\r\n");
            break;
        }
        printf("%s\r\n", buf);
        write(conn, buf, strlen(buf));
    }
    close(conn);
}

int main()
{
    int sock;
    //仍然使用socket函数创建socket，协议族指定为PF_UNIX
    if ((sock = socket(PF_UNIX, SOCK_STREAM, 0)) < 0)
        ERR_EXIT("create unix socket failed");
    
    unlink("test socket");//先删除之前留下的socket文件
    sockaddr_un servaddr;
    servaddr.sun_family = AF_UNIX; //这里指定unix域协议
    strcpy(servaddr.sun_path, "test socket"); //指定socket文件，会再目录下生成该名称的文件

    //bind
    if (bind(sock, (sockaddr*)&servaddr, sizeof(servaddr)) < 0)
        ERR_EXIT("bind failed");

    //listen
    if (listen(sock, SOMAXCONN) < 0)
        ERR_EXIT("listen failed");

    int conn;
    pid_t pid;
    while(1)
    {
        conn = accept(sock, NULL, NULL);
        if (conn == -1)
        {
            if (conn == EINTR)
                continue;
            ERR_EXIT("accept failed");
        } 
        pid = fork();
        if (pid == -1)
            ERR_EXIT("fork failed");
        if (pid == 0)//子进程
        {
            close(sock);
            handle_subprocess(conn);
            exit(EXIT_SUCCESS);
        }
        else
        {
            close(conn);
        }
    }
    close(sock);
    return 0;
}
```

## client
```cpp
#include <unistd.h>
#include <sys/un.h>
#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>

#define ERR_EXIT(m)\
    do\
    {\
        perror(m);\
        exit(EXIT_FAILURE);\
    }while(0)

int main()
{
    int sock;
    if ((sock = socket(PF_UNIX, SOCK_STREAM, 0)) < 0)
        ERR_EXIT("create unix socket failed");

    sockaddr_un servaddr;
    servaddr.sun_family = AF_UNIX; //这里指定unix域协议
    strcpy(servaddr.sun_path, "test socket");

    //start connect to the server
    if (connect(sock, (sockaddr*)&servaddr, sizeof(servaddr)) < 0)
        ERR_EXIT("connect failed");

    char buf[1024];
    while (fgets(buf, sizeof(buf), stdin) != NULL)
    {
        write(sock, buf, strlen(buf));
        read(sock, buf, sizeof(buf));//read reply from server

        printf("received from server: %s\r\n", buf);
        memset(buf, 0, sizeof(buf));
    }

    close(sock);
}
```



# 4 socketpair全双工流通道
头文件： `<sys/socket.h>`函数定义： `int socketpair(int domain, int type, int protocol, int sv[2]);`功能： 创建一个全双工的流管道，**用于父子进程间通信**； 返回值为0，失败返回-1参数：

- domain: 协议家族，指定为**PF_UNIX**
- type： 套接字类型，SOCK_STREAM流式套接字
- protocol： 协议类型
- sv： 返回的套接字对


socketpair不同与普通的pipe，是全双工的，两端可以同时发送和接收数据。代码示例如下：
```cpp
#include "common.h"

int main()
{
    int socks[2];
    if(socketpair(PF_UNIX, SOCK_STREAM, 0, socks) < 0)
        ERR_EXIT("socketpair failed");

    pid_t pid;
    pid = fork();
    if (pid > 0)
    {
        //父进程
        int val = 0;
        close(socks[1]);//只需要保留一个socket在该进程，另一个在另一进程
        while(1)
        {
            ++val;
            printf("send data: %d\r\n", val);
            write(socks[0], &val, sizeof(val));
            read(socks[0], &val, sizeof(val));
            printf("receive data: %d\r\n", val);
            sleep(1);
        }
    }
    else if (pid == 0)
    {
        int val;
        close(socks[0]);
        while(1)
        {
            read(socks[1], &val, sizeof(val));
            ++val;
            write(socks[1], &val, sizeof(val));
        }
    }
    return 0;
}
```


