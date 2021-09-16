
# 函数说明
```c
#include <execinfo.h>
/*
 * 功能描述： 获取当前堆栈地址
 * 参数说明： buffer - 函数地址数组(传出)
 *          size   - 数组的最大长度 
 * 返 回 值：实际调用的层数
 */
int backtrace(void **buffer, int size);    
/*
 * 功能描述： 将地址转换成符号字符串数组
 * 参数说明： buffer - 函数地址数组
 *          size   - 有效地址的个数 
 * 返 回 值： 符号字符串数组指针（需要调用者释放）
 */
char **backtrace_symbols(void *const *buffer, int size);
   
/*
 * 功能描述： 将地址转换成符号字符串数组并写入指定文件
 * 参数说明： buffer - 函数地址数组
 *          size   - 有效地址的个数 
 *          fd     - 文件描述符
 * 返 回 值： 无
 */
void backtrace_symbols_fd(void *const *buffer, int size, int fd);
```



# Linux示例
```c
#include<stdio.h>
#include<stdlib.h>
#include<signal.h>
#include<string.h>
#include<execinfo.h>
/*
 * 内存访问异常回调函数
 */
void segv_handle(int signum)
{
  int i;  
  size_t size;  
  int maxlen = 1024;     // 堆栈最大值  
  void *func[len];
  char **funs;  
  // 获取堆栈调用层数
  size = backtrace(func, maxlen);
  // 获取符号
  funs = (char **)backtrace_symbols(func, size);    
  // 打印输入相关信息
  fprintf(stderr, "Stack trace:\r\n");
  for (i = 0; i < size; i++)
  {
    fprintf(stderr, "%d %s \r\n", i, funs[i]);
  }
  free(funs); // 释放动态申请的内存
  exit(1);
}
//-------------------------------------------------
void func2nd()
{
  char *p=NULL;
  *p = 'A';        // 制造内存访问异常
}
void func1st()
{
  func2nd();
}
int main(const int argc,const char* argv[])
{
  // 无效内存异常信号捕获，SegmentationViolation
  signal(SIGSEGV, segv_handle);
  func1st(); 
  return 0;
}
```

编译选项：
`gcc -o BackTrace BackTrace.c -rdynamic`运行结果：
```
Stack trace:
0 ./BackTrace(segv_handle+0x54) [0x804882f] 
1 linux-gate.so.1(__kernel_sigreturn+0) [0xb77a9d1c] 
2 ./BackTrace(func2nd+0x10) [0x80488c7] 
3 ./BackTrace(func1st+0x8) [0x80488d4] 
4 ./BackTrace(main+0x28) [0x80488fe] 
5 /lib/i386-linux-gnu/i686/cmov/libc.so.6(__libc_start_main+0xf3) [0xb75f8a63] 
6 ./BackTrace() [0x8048701]
```

**-rdynamic 生成符号表，否则只有地址**：
```
Stack trace:
0 ./BackTrace() [0x80485bf] 
1 linux-gate.so.1(__kernel_sigreturn+0) [0xb77ccd1c] 
2 ./BackTrace() [0x8048657] 
3 ./BackTrace() [0x8048664] 
4 ./BackTrace() [0x804868e] 
5 /lib/i386-linux-gnu/i686/cmov/libc.so.6(__libc_start_main+0xf3) [0xb761ba63] 
6 ./BackTrace() [0x8048491]
```

需要采用addr2line命令来逐个查看：
```
# addr2line  -e BackTrace -f 0x8048657
func2nd
??:?
```
