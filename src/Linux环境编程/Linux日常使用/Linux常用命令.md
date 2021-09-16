
# 系统相关

## 1.linuxboot过程
![clip_image002.jpg](.assets/1577885926459-127114da-b954-4433-9492-33e06d411ed5.jpeg)


## 2.systemd
![clip_image004.jpg](.assets/1577885937896-02cee2ad-7cbc-4c4c-9cd6-3804ea178664.jpeg)


## 3.查看boot阶段log:dmsg命令
![clip_image006.jpg](.assets/1577885956360-13d91d50-a324-44fe-ae99-973d68b27879.jpeg)


## 4.关闭某个runlevel下自启动进程
![clip_image008.jpg](.assets/1577885970427-11ca5ec0-e237-499e-8724-62702da3afda.jpeg)


## 5.LVM
![clip_image010.jpg](.assets/1577885981649-77391762-fd6c-424b-939b-0bbaf4f97e2e.jpeg)


## 6,grub手动设置引导
![clip_image012.jpg](.assets/1577885990367-c4563b9b-3da8-4f20-b5a9-4e4bb5b2726d.jpeg)


## 7.共享库文件

共享文件是多个程序可以共同使用的代码,优点是代码重用和更新简单.

- windows:共享文件命名为*.dll
- linux:共享文件命名为Libname.so.version


Linux上共享文件保存的位置,系统变量LD_LIBRARY_PATH指定共享库的位置:•/usr/local/lib:最新版本的文件和GUN标准文件•/usr/lib:文档版本的文件和FHS相关库 •/lib:启动相关的共享库
查看某个程序使用到的共享库:![clip_image014.jpg](.assets/1577886012627-18ba95bb-b3c7-4a76-a65f-dcbfbbc54fa8.jpeg)
查看当前加载的共享库:![clip_image016.jpg](.assets/1577886022373-15e5a351-b8c1-4635-b3d0-53770f70e81c.jpeg)


# 常用命令:



## 1.Ssh-server

正确配置方法如下：
/etc/ssh/sshd_config

1. 找到`#StrictModesyes`改成`StrictModesno`（去掉注释后改成 no）
2. 找到`#PubkeyAuthenticationyes`改成`PubkeyAuthenticationyes`（去掉注释）
3. 找到`#AuthorizedKeysFile.ssh/authorized_keys`改成`AuthorizedKeysFile.ssh/ authorized_keys`（去掉注释）
4. 保存


From[http://callmepeanut.blog.51cto.com/7756998/1336864](http://callmepeanut.blog.51cto.com/7756998/1336864)


## 2.vim编译：


```
./configure --prefix=/home/barretr/tools/vim --enable-cscope --enable-pythoninterp --enable-python3interp

Make & make install
```

查看vim启动时间
v--startuptimetmp.txt
vim函数着色
VIM默认情况下，函数名是不会高亮的，将下面这段代码添加到/usr/share/vim/vim73/syntax/ c.vim文件的末尾即可：

```highlight
synmatchcFunctions"\<[a-zA-Z_][a-zA-Z_0-9]*\>[^()]*)("me=e-2

synmatchcFunctions"\<[a-zA-Z_][a-zA-Z_0-9]*\>\s*("me=e-1

hicFunctionsgui=NONEcterm=boldctermfg=blue
```

ctermfg表示函数名的颜色，可以自行更换。


## 3.VNC开启端口并设置分辨率：

vncserver-geometry1366x768


## 4.强制关闭进程：

Kill-9进程号
kill-sstop进程号----------停止进程运行,不关闭
kill-scont进程号-----------继续运行进程


## 5.systemd管理工具配置:

设置开机模式： GUI/终端模式:
`systemctlset-defaultmulti-user.target/graphical.target`
系统服务管理：
启动服务`systemctlstarthttpd.service`
关闭服务`systemctlstophttpd.service`
重启服务`systemctlrestarthttpd.service`
重新加载`systemctlreloadhttpd.service`
查看状态`systemctlstatushttpd.service`
From[http://www.linuxidc.com/Linux/2015-04/116648.htm](http://www.linuxidc.com/Linux/2015-04/116648.htm)


## 6.设置开机启动：

开机启动unit`systemctlenablepostfix.service`
开机不启动unit`systemctldisablehttpd.service`
查看开机是否启动`systemctlis-enabled.service`
From[http://www.linuxidc.com/Linux/2015-04/116648.htm](http://www.linuxidc.com/Linux/2015-04/116648.htm)


## 7.dd

dd是Linux/UNIX下的一个非常有用的命令，作用是用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换。 dd的主要选项：指定数字的地方若以下列字符结尾乘以相应的数字:
b=512,c=1,k=1024,w=2,xm=numberm
•if=file:输入文件名，缺省为标准输入。
•of=file:输出文件名，缺省为标准输出。
•ibs=bytes:一次读入bytes个字节(即一个块大小为bytes个字节)。
•obs=bytes:一次写bytes个字节(即一个块大小为bytes个字节)。
•bs=bytes:同时设置读写块的大小为bytes，可代替ibs和obs。
•cbs=bytes:一次转换bytes个字节，即转换缓冲区大小。
•skip=blocks:从输入文件开头跳过blocks个块后再开始复制。
•seek=blocks:从输出文件开头跳过blocks个块后再开始复制。(通常只有当输出文件是磁盘或磁带时才有效)
•count=blocks:仅拷贝blocks个块，块大小等于ibs指定的字节数。
•conv=conversion[,conversion...]:用指定的参数转换文件。
转换参数:
•ascii转换EBCDIC为ASCII。
•ebcdic转换ASCII为EBCDIC。
•ibm转换ASCII为alternateEBCDIC.
•block把每一行转换为长度为cbs的记录，不足部分用空格填充。
•unblock使每一行的长度都为cbs，不足部分用空格填充。
•lcase把大写字符转换为小写字符。
•ucase把小写字符转换为大写字符。
•swab交换输入的每对字节。
•noerror出错时不停止。
•notrunc不截短输出文件。
•sync把每个输入块填充到ibs个字节，不足部分用空(NUL)字符补齐。
由于dd命令允许二进制方式读写，所以特别适合在原始物理设备上进行输入/输出。例如可以用下面的命令为软盘建立镜像文件：
ddif=/dev/fd0of=disk.imgbs=1440k
有趣的是，这个镜像文件能被HD-Copy，Winimage等工具软件读出。再如把第一个硬盘的前
512个字节存为一个文件：
ddif=/dev/hdaof=disk.mbrbs=512count=1


## 8.cpio:

cpio命令主要是用来建立或者还原备份档的工具程序，cpio命令可以复制文件到归档包中，或者从归档包中复制文件。
使用示例:
•将/etc下的所有普通文件都备份到/opt/etc.cpio，使用以下命令： find/etc–typef|cpio–ocvB>/opt/etc.cpio
•将系统上所有资料备份到磁带机内，使用以下命令：
find/-print|cpio-covB>/dev/st0-------这里的/dev/st0是磁带的设备名，代表SCSI磁带机。
•查看上例磁带机上备份的文件，使用以下命令：
cpio-icdvt</dev/st0>/tmp/st_content--------有时可能因为备份的文件过多，一个屏幕无法显示完毕，此时我们利用下面命令，让磁带机的文件信息输出到文件。
•将示例1中的备份包还原到相应的位置，如果有相同文件进行覆盖，使用以下命令： cpio–icduv</opt/etc.cpio
注意，cpio恢复的路径，如果cpio在打包备份的时候用的是绝对路径，那么在恢复的时候会自动恢复到这些绝对路径下，本例就会将备份文件全部还原到/etc路径下对应的目录中。同理，如果在打包备份用的是相对路径，还原时也将恢复到相对路径下。
通过上面的示例，可以看出，cpio无法直接读取文件，它需要每个文件或者目录的完整路径名才能识别读取，而find命令的输出刚好做到了这点，因此， 命令配合使用。其实，上面的示例我们已经看到了它们的组合用法。


## 9.xargs

xargs命令是给其他命令传递参数的一个过滤器，也是组合多个命令的一个工具。它擅长将标准输入数据转换成命令行参数，xargs能够处理管道或者stdin并将其转换成特定命令的命令参数。 xargs也可以将单行或多行文本输入转换为其他格式，例如多行变单行，单行变多行。xargs的默认命令是echo，空格是默认定界符。这意味着通过管道传递给xargs的输入将会包含换行和空白，不过通过xargs的处理，换行和空白将被空格取代。xargs是构建单行命令的重要组件之一。
使用示例:
•多行变单行:
cattest.txt|xargs abcdefghijklmnopqrstuvwxyz
•管道输出转换为命令行参数: find.*.txt|xargsvim--------等同于vim1.txt2.txt......


## 10.wc:

计算文件中单词个数,返回值有三个,分别为文件行数,单词个数,字节数.
•wccities.txt
141695cities.txt


## 11.split:

分隔文件,默认按照1000行来分隔,生成xaa,xab,xac......多个文件. 常用参数:
•-a,--suffix-length=NgeneratesuffixesoflengthN(default2)
--additional-suffix=SUFFIXappendanadditionalSUFFIXtofilenames
•-b,--bytes=SIZEputSIZEbytesperoutputfile
•-C,--line-bytes=SIZEputatmostSIZEbytesoflinesperoutputfile
•-d,--numeric-suffixes[=FROM]usenumericsuffixesinsteadofalphabetic;
FROMchangesthestartvalue(default0)
•-e,--elide-empty-filesdonotgenerateemptyoutputfileswith'-n'
--filter=COMMANDwritetoshellCOMMAND;filenameis$FILE
•-l,--lines=NUMBERputNUMBERlinesperoutputfile
•-n,--number=CHUNKSgenerateCHUNKSoutputfiles;seeexplanationbelow
•-u,--unbufferedimmediatelycopyinputtooutputwith'-nr/...'


## 12.join:

将两个行数相同的文件合并成一个文件,文件需要设置好排序,一般可以用行号表示顺序,1.2.3...... 对于么有明显排序,但是行数相同的两个文件,可以使用paste命令完成合并.


## 13.expand:

将文件中tab改为空格,-t表示转换为几个空格,默认为8


## 14.设置进程优先级:

`nice-n20进程程序命令`----------设置优先级20并启动程序,其取值范围是-20至19。这个值越小，表示进程”优先级”越高，而值越大“优先级”越低
`renice-n20进程号`--------给正在运行的程序设置优先级,注意这里是进程号显示当前进程优先级:
psl


## 15.十六进制显示文件内容:

od-h文件名


## 16.fsck:

检查分区情况![clip_image018.jpg](https://cdn.nlark.com/yuque/0/2020/jpeg/690827/1577886127750-96286b21-7b53-4c46-b99f-2d399dc3aa5f.jpeg)

1. option中包含usrquotas时,表示此分区支持设置每个用户的最大配额.包含grpquota时,表示对每个用户组的配额管理重新mount(mount-oremount)之后,会显示如下: 同时,该分区下会有两个文件分别记录usr和group的配额:

![clip_image022.jpg](https://cdn.nlark.com/yuque/0/2020/jpeg/690827/1577886188818-ea6b9a74-a9b4-4e6e-ab23-2ade988c20cd.jpeg)

2. 保证quota服务在运行
2. quotaon-av,显示当前运行quota的分区
2. edquota-u用户名,设置某个用户的配额,格式如下:

![clip_image024.jpg](.assets/1577886247948-4749b7ee-ca25-4934-b010-9f3ac95afbaa.jpeg)

5. edquota-g用户组名:

![clip_image026.jpg](.assets/1577886264074-4d08452f-bb07-4f60-a1a0-ff0d8cfb080f.jpeg)

6. repquota-av,显示当前配额信息




## 17.修改用户默认的shell:

usermod用户名-szsh


## 18.设置最大登录次数:

/etc/security/limit.conf![clip_image028.jpg](.assets/1577886295251-e4e1631e-326c-434f-9de2-86dea87881de.jpeg)
ulimit设置其他限制:•-aAllcurrentlimitsarereported•-bThemaximumsocketbuffersize•-cThemaximumsizeofcorefilescreated•-dThemaximumsizeofaprocess'sdatasegment•-eThemaximumschedulingpriority("nice")•-fThemaximumsizeoffileswrittenbytheshellanditschildren•-iThemaximumnumberofpendingsignals•-lThemaximumsizethatmaybelockedintomemory•-mThemaximumresidentsetsize(manysystemsdonothonorthislimit)•-nThemaximumnumberofopenfiledescriptors(mostsystemsdonotallowthis valuetobeset)•-pThepipesizein512-byteblocks(thismaynotbeset)•-qThemaximumnumberofbytesinPOSIXmessagequeues•-rThemaximumreal-timeschedulingpriority•-sThemaximumstacksize•-tThemaximumamountofcputimeinseconds•-uThemaximumnumberofprocessesavailabletoasingleuser•-vThemaximumamountofvirtualmemoryavailabletotheshelland,onsome systems,toitschildren•-xThemaximumnumberoffilelocks•-TThemaximumnumberofthreads ============================================================


## 19.TCP
![clip_image030.jpg](.assets/1577886345314-55fe1372-2f84-4129-80fe-04fcb259918e.jpeg)


## 20.UDP
![clip_image032.jpg](.assets/1577886355085-359ec21f-d031-4c5b-8f92-22f6694eca73.jpeg)


## 21.ICMP
![clip_image034.jpg](.assets/1577886364781-d6a42d82-1f54-46fb-9558-43b325dd21ef.jpeg)


## 22.IPV6vsIPV4
![clip_image036.jpg](.assets/1577886380223-c8db4a68-d2cb-48df-b3fd-5a67f6fa8a83.jpeg)


## 23.网络配置命令

•ifdown/ifup网卡名:关闭开启网卡
•ipaddressshow:显示网络配置信息,类似ifconfig
•ipaddressadd1.1.1.1/8dev网卡:添加ip地址到某网卡
•route:显示ip路由表
•routeadddefaultgw192.168.1.1:添加路由项.另一种形式如下:![clip_image040.jpg](.assets/1577886403894-461d5f77-4b8a-4ed9-86a6-3a87864b6c45.jpeg)
•dig网址:显示网址对应的解析ip
/etc/nsswitch.conf中保存了解析网址的顺序,默认是filesdns.即先查找hosts文件,后查询dns服务器.可以修改该文件改变顺序![clip_image042.jpg](.assets/1577886417942-e6635391-9832-4a3f-9782-4477d2fc1225.jpeg)


## 24.时间日期设置:

•date:显示当前时间,可以添加%*参数,获取指定格式的时间
•date-s“1/1/20183:00pm”:设置当前时间日期
•timedatectl:显示时间,日期,时区等所有信息
•hwclock:显示硬件时间
•hwclock-s:自动设置软件时间,取值于硬件时间
•hwclock-w:自动设置硬件时间,取值于date时间
•hwclock--set--date="01/01/20183:00pm":手动设置硬件时间
•ln-s/usr/share/zoneinfo/Asia/Shanghai/etc/localtime:设置当前时区./usr/share/ zoneinfo保存所有时区的信息,添加软连接即可.
•tzselect:设置时区,采用选择方式,比上面软连接方便.


## 25 设置代理

有些linux服务器处于内网，并且没有公网ip，故要想与外网进行http/https通信只能通过nat或者加proxy的方式。nat服务器有网段的限制，而http/https proxy代理则没有，使用起来也方便。
linux系统设置http/https proxy的方法，在**/etc/bashrc**或者**/etc/profile**中添加如下环境变量：

```
export http_proxy=http://1.1.1.1:8082
export https_proxy=http://1.1.1.1:8082
```



### 单独设置apt代理

临时代理：`sudo apt -o Acquire::http::proxy="http://barretr:passwd@135.251.3xx.16:80" update`永久代理：

```shell
sudo su
vi /etc/apt/apt.conf.d/10proxy   #这个文件正常不存在，会新建一个
#编辑内容为：
Acquire::http::Proxy "http://user:pwd@192.168.1.1:8080";
```



## 26 关机或重启时提示"A stop job is running for ..."

编辑/etc/systemd/system.conf
修改下面两个变量为：

```
DefaultTimeoutStartSec=10s
DefaultTimeoutStopSec=10s
```

执行：`systemctl daemon-reload`


## 27 用户不在sudoers中

打开/etc/sudoers文件，在root之后添加对应用户名，并指定权限，例如：`barret ALL=(ALL:ALL) ALL`


## 28 tmux编译


```shell
sh autogen.sh
./configure --prefix=/home/barretr/tools/tmuxN LIBEVENT_CFLAGS="-I/home/barretr/tools/Libevent/include" LIBEVENT_LIBS="-L/home/barretr/tools/Libevent/lib -levent"
make
make install
```

---


