Oprofile是linux上的性能监测工具，通过CPU硬件提供的性能计数器对事件进行采样，**从代码层面分析程序的性能消耗情况**，找出程序性能的问题点。

# 1 oprofile安装
RHEL和ubuntu上直接提供了安装包，可以直接安装。
```bash
$sudo apt install oprofile
$yum install oprofile
```
> 


# 2 命令选项

## 2.1 设置检测事件
`opcontrol --event=<event-name>:<sample-rate>:<unit-mask>:<kernel>:<user>`
各字段含义如下：

| event-name | 事件名称，如CPU_CLK_UNHALTED，支持的事件列表可以通过**opcontrol --list-events命令列出** |
| --- | --- |
| sample-rate | 触发一次采样的事件计数值，如100000 |
| unit-mask | 硬件单元掩码，如0x0F |
| kernel | 是否对内核进行监测，取值0或1 |
| user | 是否对用户空间进行监测，取值0或1 |


使用示例：`opcontrol --event=CPU_CLK_UNHALTED:6000:0x00:0:1       #对core cycle进行profile`
> 


## 2.2 开始检测
```bash
opcontrol --init   #加载模块
opcontrol --reset  #清除当前会话中的数据
opcontrol --start  #开始profiling
```
> 
> 

注：如果opcontrol –start报错，可以使用以下命令解决
```bash
opcontrol --deinit
echo 0 > /proc/sys/kernel/nmi_watchdog
```
> 
> 


## 2.3 保存数据
程序业务执行完成后，使用下面的命令将监测数据转储下来：`opcontrol --dump #把收集到的数据写入文件`
> 


## 2.4 停止检测
```bash
opcontrol –stop #停止profiling
opcontrol –shutdown #关闭守护进程oprofiled
opcontrol –deinit #卸载模块
```

## 2.5 其他选项
除了监测事件以外，oprofile还提供了以下的用途：

### 是否监测内核文件
当要对内核进行监测时，需要指定内核的文件路径，使用下面的命令（其中file为内核文件完整路径）：`opcontrol --vmlinux=file`
> 

如果不需要对内核进行性能监测，则此处直接使用--no-vmlinux参数：`opcontrol --no-vmlinux`
> 


### 设置监测数据分离方式
`opcontrol --separate=<choice>`<choice> 有几下几个参数：

| **none** | do not separate the profiles （default） |
| --- | --- |
| **library** | generate per-application profiles for libraries |
| **kernel** | generate per-application profiles for the kernel and kernel modules |
| **all** | generate per-application profiles for libraries and per-application profiles for the kernel and kernel modules |






# 3 性能数据分析
监测数据转储后，即可以通过`opreport`和`opannotate`命令查看性能消耗情况了。1） 显示各模块整体的性能消耗情况：`opreport`
> 

2） 显示各函数的性能消耗情况：`opreport -l`
> 

3） 显示代码级别的性能消息情况：`opannotate -s`**

# 4 配置脚本


```bash
LOGFILE="/tmp/OProfile_LOG"
DURATION=600
echo "start OProfile ($DURATION s)"
echo "set up OProfile"
opcontrol --deinit 2>> $LOGFILE  -- 加载
modprobe oprofile timer=1  
opcontrol --setup --vmlinux=/usr/lib/debug/lib/modules/`uname -r`/vmlinux  -- 设置跟踪vmlinux
opcontrol -c 0
opcontrol –reset  --清空当面的信息
echo "start OProfile" 
opcontrol --start >> $LOGFILE 2>&1   -- 开始跟踪，中间执行应用程序
sleep $DURATION
echo "shutdown OProfile"
opcontrol --shutdown >> $LOGFILE 2>&1  -- 结束跟踪
echo "report OProfile"
opreport -l > $OUTDIR/opreport_-l.txt 2>> $LOGFILE  --以函数的角度显示检测结果
echo "oprofile temp data size: $(du -sh /var/lib/oprofile)"
echo "cleanup OProfile"
opcontrol --reset
opcontrol --deinit 2>> $LOGFILE
echo "finish OProfile"
```


