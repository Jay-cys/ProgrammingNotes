
# 1 互斥锁

## 1.1 互斥锁函数

- pthread_mutex_init：创建互斥锁
- pthread_mutex_lock：锁定
- pthread_mutex_unlock：解锁
- pthread_mutex_destroy：删除互斥锁




## 1.2 互斥锁示例
```cpp
#include <pthread.h>
#include <errno.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <stdlib.h>
#include <stdio.h>

int run_now=1;  /*用run_now代表共享资源*/
pthread_mutex_t work_mutex;  /*定义互斥量*/

void *thread_function(void *arg)
{
    sleep(1);
    
    /*对互斥量加锁*/
    if(pthread_mutex_lock(&work_mutex)!=0)
    {
        perror("Lock failed");
        exit(1);
    }
    else
        printf("function lock\n");

    for (int i=0; i<5; i++)
    {
        /*分线程：如果run_now为1就把它修改为1*/
        if(run_now==2)  
        {
            printf("function thread is run\n");
            run_now=1;
        }
        else
        {
            printf("function thread is sleep\n");
            sleep(1);
        }
    }
    
    /*对互斥量解锁*/
    if(pthread_mutex_unlock(&work_mutex)!=0)  
    {
        perror("unlock failed");
        exit(1);
    }
    else
        printf("function unlock\n");

    pthread_exit(NULL);
}

int main()
{
    pthread_t a_thread;
     
    if(pthread_mutex_init(&work_mutex,NULL)!=0)  /*初始化互斥量*/
    {
        perror("Mutex init faied");
        exit(1);
    }

    if(pthread_create(&a_thread,NULL,thread_function,NULL)!=0)  /*创建新线程*/
    {
        perror("Thread createion failed");
        exit(1);
    }

    if(pthread_mutex_lock(&work_mutex)!=0)  /*对互斥量加锁*/
    {
        perror("Lock failed");
        exit(1);
    }
    else
        printf("main lock\n");
     

    for (int i=0; i<5; i++)
    {
        if(run_now == 1)  /*主线程：如果run_now为1就把它修改为2*/
        {
            printf("main thread is run\n");
            run_now=2;
        }
        else
        {
            printf("main thread is sleep\n");
            sleep(1);
        }
    }

    if(pthread_mutex_unlock(&work_mutex)!=0) /*对互斥量解锁*/
    {
        perror("unlock failed");
        exit(1);
    }
    else
        printf("main unlock\n");

    pthread_join(a_thread,NULL); /*等待子线程结束*/
    pthread_mutex_destroy(&work_mutex); /*收回互斥量资源*/

    exit(0);
}
```



# 2 自旋锁
当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环。**减少了不必要的上下文切换，但会加剧CPU消耗**。

- pthread_spin_init
- pthread_spin_destroy
- pthread_spin_lock
- pthread_spin_unlock




# 3 读写锁

- 读锁之间是共享的：即一个线程持有了读锁之后，其他线程也可以以读的方式持有这个锁
- 写锁之间是互斥的：即一个线程持有了写锁之后，其他线程不能以读或者写的方式持有这个锁
- 读写锁之间是互斥的：即一个线程持有了读锁之后，其他线程不能以写的方式持有这个锁



- Pthread_rwlock_init
- Pthread_rwlock_destory
- Pthread_rwlock_rdlock
- Pthread_rwlock_wrlock
- Pthread_rwlock_unlock




# 4 POSIX条件变量
当一个线程互斥的访问某个变量时，需要等待其他线程改变变量状态之后才能访问。这样的变量要设置成**条件变量**。头文件
```cpp
#include <pthread.h>
```



## 4.1 条件变量函数

- pthread_cond_init：初始化条件变量
- pthread_cond_destroy：删除条件变量
- pthread_cond_wait：在一个指定的条件上等待。具体执行逻辑如下
   - 对第二个参数的互斥锁解锁
   - 等待条件，知道有线程发起条件满足通知
   - 对互斥锁重新加锁
- pthread_cond_signal：当条件满足时，向指定线程发送通知。如果没有线程wait，通知被丢弃。
- pthread_cond_broadcast：当条件满足时，向所有线程发送通知


**条件变量通常与互斥锁配合使用**，条件变量使用规范如下：

- 等待条件代码
```cpp
pthread_mutex_lock(&mutex); //锁定互斥量
//这里用while是为了防止虚假唤醒，如果是虚假唤醒,条件并没有改变,需要用while再次判断条件是否满足
while(条件为假) 
    pthread_cond_wait(cond,mutex);
修改条件
pthread_mutex_unlock(&mutex);
```

- 给线程发送信号代码
```cpp
pthread_mutex_lock(&mutex);
设置条件为真
pthread_cond_signal(cond);
pthread_mutex_unlock(&mutex);
```

## 4.2 生产者消费者问题示例
```c
#include <unistd.h>
#include <sys/types.h>
#include <pthread.h>
#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <string.h>
 
#define ERR_EXIT(m) \
	do \
	{	\
		perror(m); \
		exit(EXIT_FAILURE); \
	}while(0)
	
#define CONSUMERS_COUNT 2
#define PRODUCERS_COUNT 4
 
pthread_cond_t g_cond;
pthread_mutex_t g_mutex;
 
// 创建的线程ID保存在g_thread中
pthread_t g_thread[CONSUMERS_COUNT+PRODUCERS_COUNT];
 
int nready=0;
 
///消费者
void* consume(void* arg)
{
    //most platforms pointers and longs are the same size, but ints and pointers often are not the same size on 64bit platforms
	int num = (long)arg;
	while(1)
	{
		pthread_mutex_lock(&g_mutex);
		while(nready == 0)
		{
			printf("(%d)begin wait a condition....\n",num);
			pthread_cond_wait(&g_cond,&g_mutex);
		}
		printf("(%d) end wait a condition....\n",num);
		printf("(%d) begin consume product ....\n",num);
		--nready;
		printf("(%d) end consume product ....\n",num);
		pthread_mutex_unlock(&g_mutex);
		sleep(1);
	}
	return NULL;
}
 
//// 生产者
void* produce(void* arg)
{
    //most platforms pointers and longs are the same size, but ints and pointers often are not the same size on 64bit platforms
	int num = (long)arg;
	while(1)
	{
		pthread_mutex_lock(&g_mutex);
		printf(" (%d) begin produce product ...\n",num);
		++nready;
		printf(" (%d) end produce product....\n",num);
		pthread_cond_signal(&g_cond);
		printf(" (%d) signal \n",num);
		pthread_mutex_unlock(&g_mutex);
		sleep(5);
	}
	return NULL;
}
 
int main(void )
{
	//初始化互斥锁
	pthread_mutex_init(&g_mutex,NULL);
	//初始化条件变量
	pthread_cond_init(&g_cond,NULL);
	
	/// 创建消费者线程
	for(int i=0; i<CONSUMERS_COUNT; i++)
		pthread_create(&g_thread[i],NULL,consume,(void*)i);
    
	sleep(1);
	/// 创建生产者线程
	for(int i=0; i<PRODUCERS_COUNT; i++)
		pthread_create(&g_thread[CONSUMERS_COUNT+i],NULL,produce,(void*)i);
    
	// 等待线程的结束	
	for(int i=0; i<CONSUMERS_COUNT+PRODUCERS_COUNT; i++)
		pthread_join(g_thread[i],NULL);
		
	//销毁互斥锁和条件变量	
	pthread_mutex_destroy(&g_mutex);
	pthread_cond_destroy(&g_cond);
	
	return 0;
}
```



# 5 信号量
