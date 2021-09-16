
# 1 基本命令

## 1.1设置断点
| break 函数名break 行号break 类名：函数名 | Set a breakpoint at specified function or line number. |
| --- | --- |
| break +偏移量break -偏移量 | Set a breakpoint at specified number of lines forward or backward from current line of execution. |
| break 文件名：函数名 | Set a breapoint at specified funcname of given filename. |
| break 文件名：行号 | Set a breakpoint at given line-no of specified filename. |
| break *地址 | Set a breakpoint at specified instrunction address. |
| break 断定 if 条件 | 当条件为true时，暂停运行 |
| break line thread thread-no | Set a breakpoint at line in thread with thread-no. |
| tbreak | tbreak is similar to break but it will temporary breakpoint. i.e. Breakpoint will be deleted once it is hit. |
| info break | 查看当前设置的断点 |


## 1.2 监视点
用于监视某个变量在什么地方被修改，**可能会降低运行速度**。

| watch 表达式 或 变量 | 设置监视点，在表达式或变量发生变化时暂停 |
| --- | --- |
| awatch 表达式 或 变量 | 被访问、修改时暂停 |
| rwatch 表达式 或 变量 | 被访问时暂停 |
| info watchpoints | 查看当前监视点 |


## 1.3 删除或使能断点监视点
| clearclear functionclear line-number | Delete breakpoints as identified by command option.Delete all breakpoints in functionDelete breakpoints at given line |
| --- | --- |
| delete | Delete all breakpoints, watchpoints, or catchpoints. |
| delete breakpoint-numberdelete range | Delete the breakpoints, watchpoints, or catchpoints specified by number or ranges. |
| disable breakpoint-numberdisable rangeenable breakpoint-numberenable range | Enable/Disable breakpoints, watchpoints or catchpoints specified by number or ranges. |
| enable breakpoint-number once | Enables given breakpoint once. And disables it after it is hit. |


## 1.4 显示调用栈
| backtrace 或 bt | 显示所有调用栈 |
| --- | --- |
| bt n | 显示开头N个调用栈 |
| bt -n | 显示最后n个调用栈 |
| frame n | 查看指定的栈位置，可以打印此位置上的code和变量 |


## 1.5 打印变量
| print var 或 p | Prints the current value of the variable "var" |
| --- | --- |


## 1.6 显示寄存器
| info registersi r | Display processor's register contents |
| --- | --- |
| p/格式 $寄存器名 | 打印寄存器的值，可以定义不同格式 |
| x/格式 地址或$寄存器 | 显示内存内容，可以使用如下格式 |


可以使用的格式如下：

| 格式 | 说明 |
| --- | --- |
| x | 十六进制 |
| d | 十进制 |
| u | 无符号十进制 |
| o | 八进制 |
| t | 二进制 |
| a | 显示地址 |
| c | 显示为字符 |
| f | 浮点小数 |
| s | 显示为字符串 |
| i | 显示为机器语言，用于x命令 |


## 1.7 单步调试
| continue c | 继续运行，遇到断点或到达程序结尾停止 |
| --- | --- |
| continue 次数 | Continue but ignore current breakpoint  number times. |
| finish | Continue until the current function is finished |
| step | 单步运行，进入函数内部 |
| step N | Runs the next N lines of program |
| next | 单步运行，不进入函数内部 |
| nexti 和 stepi | 单步调试汇编指令 |
| set var=val | 改变变量的值 |
| q | Quit from gdb |


## 1.8 生成core dump
| generate-core-file | 生成当前调试进程的core dump |
| --- | --- |


## 1.9 info命令
| info line         | 当前调试位置的程序的地址 |
| --- | --- |
| info locals       | 当前堆栈下的局部变量 |
| info macro xx     | 显示宏xx的值 |
| info program      | 程序当前的执行状态 |
| info registers    | 列出整形寄存器和值 |
| info set          | 显示GDB所有的设置（内容较多） |
| info sharedlibrary | 显示加载的共享库的状态 |
| info source       | 当前源文件的信息 |
| info sources      | 程序里的源文件信息 |
| info stack        | 程序堆栈的回溯信息（类似backtrace） |
| info target       | 当前被调试文件的一些信息（可以看到ELF各个段） |
| info threads      | 当前已知的线程信息 |
| info tracepoints  | 追踪点的状态（how to use） |
| info types        | 显示所有定义的数据类型（例如你定义的uchar的实际类型） |
| info variables    | 所有全局的和静态的变量信息 |


# 2 高级技巧

## 2.1 attach运行中的进程
要调试守护进程等已经启动的进程，或陷入死循环无法返回的进程，可以用attach命令：

| attach pid | attach到进程ID为pid的进程上 |
| --- | --- |
| detach | 分离gdb中的进程，进程可以继续运行 |
| info proc | 查看进程信息 |


## 2.2 断点命令
定义在到达断点后自定执行的命令，格式如下：
```
commands 断点编号
	命令...
end
```
使用示例：
```
break foo if x>0
commands
    silent
    p "x is %d\n", x
    cont
end
```

## 2.3 值的历史
| **变量** | **说明** |
| --- | --- |
| $ | 值历史的最后一个打印值 |
| $n | 值历史的第n个值 |
| $$ | 值历史的倒数第二个值 |
| $$n | 值历史的倒数第n个值 |
| $_ | x命令显示的最后地址 |
| $__ | x命令显示的最后地址的值 |
| $_exitcode | 调试的程序的返回代码 |
| $bpnum | 最后设置的断点编号 |



