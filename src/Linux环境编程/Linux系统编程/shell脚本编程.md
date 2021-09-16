
# 1. Shell脚本基础

把一系列的Linux命名放在一个文件中, 下面为shell的脚本的格式:

```shell
#！/bin/bash shell脚本的起始符号，指定脚本解释器
//Linux命令集
Cd #shell注释
..bash_profile
Date
who
```

Shell脚本中反引号`允许将shell命令的输出赋值给变量。Linux下出来输入和输出重定向还有内联输入重定向，下面是其示例样式：

```shell
wc << EOF
> test string 1
> test string 2
> test string 3
> EOF
```


## 1.1 数学计算

Bash中可以使用如下方式将数学运算包括起来：`[数学表达式]`或`((数学表达式))`

Bash默认只支持整数运算，要进行浮点数运算需要使用bc（bash内建的计算器: `变量=echo "bc选项；数学表达式"| bc`

第二种方式是使用内联重定向，格式见示例：

```shell
#!/bin/bash
#shell中的数学计算和浮点数运算解决方案
var1=100
var2=50
var3=45
var4=$[$var1*($var2-$var3)] #常规整数运算
echo "整数运算的输出结果：$var4"

var5=`echo "scale=4;$var1/$var3"|bc` #echo 格式的浮点数运算
echo "浮点数echo解决方案的输出结果：$var5"

var6=`bc << EOF #内联重定向的浮点数运算
scale=4
$var1/$var3
EOF
`
echo "浮点数内联重定向解决方案的输出结果：$var6"
```


## 1.2 条件分支语句

Shell中if-else命令和python等脚本的类似，但是if或elif的条件是**命令执行的状态而不是一个等式的真假**，这是要注意的地方。下面是if条件分支的示例：

```shell
#!/bin/bash
#if-then-else条件分支示例
testuser=badtest 
if grep $testuser /etc/passwd;then
	echo "该用户主目录下的文件有："
	ls -a /home/$testuser/.b*
#若还有条件可以用elif
else
	echo "用户$testuser在系统中不存在"
fi
```

If只能用命令执行结果作为判断依据，如果要进行数的比较怎么办？这时应该使用**test**命令。Test命令可以进行数字，字符串，文件等多种比较，为真则返回状态码0,从而进行if操作。在if语句中可以用**test 条件语句**或**[条件语句]**来使用test比较。

| **数值比** |  |
| --- | --- |
| N1   -eq n2 | N1,n2是否相等 |
| N1   -ge n2 | N1是否大于等于n2 |
| N1   -gt n2 | N1是否大于n2 |
| N1   -le n2 | N1是否小于等于n2 |
| N1   -lt n2 | N1是否小于n2 |
| N1   -ne n2 | N1是否不等于n2 |
| **字符串比较** |  |
| Str1   = str2 | 是否相等（！=） |
| Str1   < str2 | Str1是否比str2小（>）,注意需要\\进行转义操作 |
| -n   str1 | Str1的长度是否非0 |
| -z   str1 | Str1长度是否为0 |
| **文件比较** |  |
| -d  file | File是否存在且是一个目录 |
| -e  file | File是否存在 |
| -f  file | File是否存在且是一个文件 |
| -r  file | File是否存在可读（-w，-x） |
| -s  file | File是否存在且非空 |
| -O  file | File是否存在并属于当前用户所有 |
| -G  file | File是否存在并默认组和当前用户相同 |
| File1   -nt  file2 | File1是否比file2新 |
| File1   -ot  file2 | File1是否比file2旧 |


除了使用test表示条件还有两种bash新增的扩展：

- ((表达式))：数学比较，可以使用标准的常见比较字符和运算，并且不需要进行转义
- [[表达式]]：字符串比较，使用test命令的比较，并且还可以只用正则表达式


## 1.3 循环命令

第一种循环是for循环，它有一个in列表，循环会依次取列表中的值进行操作。In列表可以直接添加，以空格为分割（可以**IFS=$**'  '指定别的分隔符），也可以从变量读取或者从命令读取（需要'  '）。

```shell
for var in Alabama Alaska Arizona Arkansas California Colorado "new York"；do echo 	the next state is $var
done
"C语言风格的for循环
for (( i=1;i<=10;i++));do 
	echo "the next number is $i" 
done
```

While循环是另一种常用的循环方式，除了使用test进行条件判断，还可以指定多个命令，最后一个命令的状态决定循环是否继续：

```shell
while [ $var1 -gt 0 ];do 
	echo $var1 
	var1=$[ $var1-1 ]
done
"while可以有多个命令，一个命令一行，最后一个测试命令状态决定循环是否继续
var2=10
while echo $var2 [ $var2 -gt 0 ];do 
	echo this is inside the loop
	var2=$[$var2-1]
done
```

Until命令和while格式相同，工作方式相反，until要求指定一个状态非0的命令或比较，只有条件非0才会执行循环内容。

在done之后加一个输出重定向>可以将循环的输出重定向到一个文件中，也可以用管道|作为参数传递给另外一个shell命令。


## 1.4 处理用户输入

第一种用户输入是使用位置参数，位置参数变量是标准的数字，**![](https://g.yuque.com/gr/latex?0**%E8%A1%A8%E7%A4%BA%E7%A8%8B%E5%BA%8F%E5%90%8D%EF%BC%8C**#card=math&code=0%2A%2A%E8%A1%A8%E7%A4%BA%E7%A8%8B%E5%BA%8F%E5%90%8D%EF%BC%8C%2A%2A&height=24&width=151)#**表示参数的个数，**$1-![](https://g.yuque.com/gr/latex?9**%E4%B8%BA%E6%89%A7%E8%A1%8C%E8%84%9A%E6%9C%AC%E5%91%BD%E4%BB%A4%E6%97%B6%E8%BE%93%E5%85%A5%E7%9A%84%E5%8F%82%E6%95%B0%EF%BC%8C%E6%95%B0%E5%AD%97%E5%A4%A7%E4%BA%8E9%E6%97%B6%E9%9C%80%E8%A6%81%E5%8A%A0%7B%7D%EF%BC%8C%E5%A6%82#card=math&code=9%2A%2A%E4%B8%BA%E6%89%A7%E8%A1%8C%E8%84%9A%E6%9C%AC%E5%91%BD%E4%BB%A4%E6%97%B6%E8%BE%93%E5%85%A5%E7%9A%84%E5%8F%82%E6%95%B0%EF%BC%8C%E6%95%B0%E5%AD%97%E5%A4%A7%E4%BA%8E9%E6%97%B6%E9%9C%80%E8%A6%81%E5%8A%A0%7B%7D%EF%BC%8C%E5%A6%82&height=24&width=417){10}。位置参数可以加入任意多个命令行参数。另外，![](https://g.yuque.com/gr/latex?*%E5%92%8C#card=math&code=%2A%E5%92%8C&height=24&width=24)@会存储所有的数据，不同的是**![](https://g.yuque.com/gr/latex?%5C***%E5%B0%86%E6%95%B0%E6%8D%AE%E4%BD%9C%E4%B8%BA%E5%8D%95%E4%B8%AA%E5%AD%97%E7%AC%A6%E4%B8%B2%E4%BF%9D%E5%AD%98%EF%BC%8C**#card=math&code=%5C%2A%2A%2A%E5%B0%86%E6%95%B0%E6%8D%AE%E4%BD%9C%E4%B8%BA%E5%8D%95%E4%B8%AA%E5%AD%97%E7%AC%A6%E4%B8%B2%E4%BF%9D%E5%AD%98%EF%BC%8C%2A%2A)@**作为多个独立的单词保存可以进行循环遍历。

```shell
#!/bin/bash
#位置参数，允许用户输入，在运行脚本同时指定参数
total=$[ $1*$2 ]
echo "第一个参数为：$1" echo "第二个参数为：$2"
echo "两者的乘积为：$total" name=`basename $0`   #basename命令可以返回程序名而去掉路径 
echo "程序名为：$name"
```

Shift命令默认会将每个参数减一，即$2变为$1，$1删除，可以用作参数遍历。Shift还可以指定移动的位数，默认为1,可以改变为其他值：**shift**  **2**。

当脚本同时有option和参数，并且选项又带有参数，这时候分离选项和参数就很重要。在shell中getopts可以将这一过程简单化。Getopts的格式为：**getopts  optstring vatiable**。Optstring为有效的选项字母，若选项带有参数则在字母后加冒号；若要去掉错误信息则在optstring前加冒号。**![](https://g.yuque.com/gr/latex?OPTARG**%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8F%E4%BF%9D%E5%AD%98%E9%80%89%E9%A1%B9%E5%8F%82%E6%95%B0%E5%80%BC%EF%BC%9B**#card=math&code=OPTARG%2A%2A%E5%85%A8%E5%B1%80%E5%8F%98%E9%87%8F%E4%BF%9D%E5%AD%98%E9%80%89%E9%A1%B9%E5%8F%82%E6%95%B0%E5%80%BC%EF%BC%9B%2A%2A&height=24&width=312)OPTIND**全局变量保存参数列表中getopts正在处理的参数的位置。下面是示例:

```shell
while getopts :ab:c opt;do 
case "$opt" in
	a)	echo "执行a选项操作";;
	b)	echo "执行b选项操作，并有参数$OPTARG";;#获取选项参数
	c) echo "执行c选项操作";;   #有多个选项参数同样访问$OPTARG，会自动区分
	*) echo "未知的选项：$opt";;
esac done shift $[ $OPTIND -1 ]  #shift和$OPTIND结合来移动参数 
count=1 for param in $@;do 
	echo "位置参数 $count:$param" 
	count=$[$count+1]
done
```

除了在执行脚本是指定参数和选项，同样可以在脚本执行过程中获取用户的输入。**Read**  变量命令可以从键盘或文件接受输入，并存入一个标准变量。下面是read命令在脚本中常见用法：

```shell
read -p "输入你的名字：" name  #-p为打印输入提示
read -p "输入你的英文名：" first last  #接受多个输入 
read -t 5 -p "输入一个名字：" name2  #-t 指定超时时间（秒）
read -n 1 -p "继续吗？[Y/N]" answer  #-n指定接受的字符个数
read -s -p "输入你的密码：" passwd  #隐藏输入内容
#read从文件读入
count=1 cat readfile | while read line;do 
	echo "行数$count:$line"
	count=$[ $count+1]
done
```


## 1.5 数据呈现

在脚本中重定向输出有两种：临时重定向和永久重定向。

- 临时重定向：`Echo “this is an error message” >&2 **#**直接将要重定向的内容定向到对应文件描述数字`
- 永久重定向：`exec 1>file #将标准输出重定向到文件，会对所有标准输出起作用`

永久重定向后，有时候我们需要将输出再变为原来的输出，可以使用下面方法：自定义一个文件描述符指向原来输出，当需要变为原来输出时只要再指向自定义的即可。同样重定向输入也是这样:

```shell
"恢复输出：
exec 3>&1 exec 1>file   exec 1>&3
exec 3>&-   #关闭文件描述符  
"恢复输入:
exec 4<&0 exec 0<file
exec 0<&4   Exec   4>&-
```

如果要阻止命令输出或清空文件内容，可以将输出或文件内容重定向到/dev/null件，null的任何数据都不会保存，相当于被丢弃了。
Linux系统的/tmp目录可以存放临时文件，系统可以在启动时自动清理。下面介绍临时文件的创建使用。

```shell
tempfile=`mktemp test16.XXXXXX` #创建临时文件，-t 在/tmp下建立文件；-d 建立目录
exec 3>$tempfile  #自定义输出到临时文件
echo "this script writes to temp file $tempfile"
echo "this is the first line">&3 exec 3>&echo "Done creating temp file.The contents are:"
cat $tempfile    #查看临时文件内容 rm -f $tempfile  #删除临时文件
```


# 2. 高级shell脚本


## 2.1 脚本控制

捕捉信号：**Trap command signals**命令在脚本中捕捉信号，脚本受到trap中列出的信号，trap会阻止shell脚本执行它，而是执行command命令。当command为-时，则表示移除后面的信号。下面是trap的示例：

`trap "echo 'sorry! I have trapped Ctrl-C'" SIGINT SIGTERM`

后台运行脚本：在运行脚本时后面加上**&**符号可以后台执行，shell可以进行其他操作或者运行多个后台作业。

非终端运行脚本：**nohup ./**脚本 **&**。后台运行，即使终端退出也继续执行，输出内容存放在nohup.out文件中。

**作业控制**：**Jobs**：列出当前分配给shell的作业，带+的为默认的作业。**Bg**  **作业号**：重启停止的作业，后台执行。**Fg** **作业号**：重启作业，前台执行。

**优先级调整**：脚本启动默认优先级是0（-20～20）。**Nice -n 10 ./脚本**：nice命令启动脚本时指定优先级。**Renice 10 p** **进程号**：调整正在运行的脚本的优先级。

**定时运行作业**：**at -f** **脚本名** **time**：在特定时间执行脚本，脚本的输出会提交到用户的e-mail，可以用mial命令查看。**Atq** :列出作业队列在等待的作业。**Atrm** **作业号**：删除等待中的作业。**Min hour dayofmonth month dayogweek command**：使用cron时间表定时执行command命令。

另外在/etc下存在cron的目录，可以将需要定时执行的脚本复制到相应的cron目录中。


## 2.2 shell函数

Shell脚本中使用**function**关键字定义函数，使用**echo**返回值。给函数传递参数和给脚本传递参数一样，使用$n来获取参数，但是函数不能直接获取脚本参数，要传递给函数：

```shell
function addem {
if [ $# -eq 0 ] || [ $# -gt 2 ];then
	echo -1
elif [ $# -eq 1 ];then 
	echo $[ $1 + $1 ] 
else 
	echo $[ $1 + $2]
fi
} 
echo -n "10加15等于：" value=`addem 10 15` #传递参数给函数，并接受函数返回值
echo $value
echo -n "只输入一个数10："
value=`addem 10` echo $value
echo -n "不输入参数："
value=`addem`
echo $value
```

函数也可以传递数组参数，但是需要将数组变了分解后再作为函数参数使用，不然只能得到数组的第一个值，方法如下：

```shell
function array {
	local newarray   #local表示该变量只能在函数内使用
	newarray=`echo "$@"` 
	echo "The new array value is:${newarray[*]}" #返回数组值
}
myarray=(1 2 3 4 5) 
echo "The original array is ${myarray[*]}"
array ${myarray[*]}  #传递数组参数
```

Shell还支持函数库文件，把常用的shell函数放在同一个文件中作为库函数，需要使用时通过source命令（即点操作符）倒入库文件引用函数。

```shell
. ./funcLab  #source命令，注意库文件路径
value1=10 value2=5
result1=`addem $value1 $value2`
echo "相加：$result1"
```


## 2.3 图形化脚本

Shell最简单的是**文本菜单**，可以使用echo显示各个选择项，也可以使用select自动生成菜单，然后在case语句中判断选择执行不同的函数和命令：

```shell
select option in "Display disk space" "Display logged on users" "Display memory usage" "Exit program";do case $option in
"Exit program") break;;
"Display disk space") diskspace;;
"Display logged on users") whoseon;;
"Display memory usage") menusage;;
*) clear    
echo "sorry,wrong selection";; esac done
```

**Dialog**包可以在文本环境用ANSI转义控制字符来创建标准的窗口对话框。下面演示如何在shell脚本中使用dialog。Dialog的标准格式为：`dialog --控件名称 控件参数`。下面是常用的几个控件，其他可以在dialog帮助中查看：

```shell
dialog --title Msgbox --msgbox "this is a msgbox" 10 20  #消息对话框
dialog --title Yesno  --yesno "this is a yesno" 10 20   #yes/no对话框
#输入对话框,注意输入会输出到stderr，需要重定向来获取
dialog --title inputBox --inputbox "Enter your name:" 10 20 dialog --title textbox --textbox /etc/passwd 15 45  #文本框，显示大量文本
#menu，创建窗口菜单,选项同样输出到stderr
dialog --title menu --menu "Sys Admin menu" 20 30 10 1 "Display disk space" 2 "Display users" 3 "exit program"
dialog --title fselect --fselect $HOME/ 10 50  #fselect，浏览文件并选择
```

除了使用基本的dialog包，在gnome和KDE桌面环境下有各自的扩展dialog包：**zenity**和**Kdialog**，下面简单介绍zenity。Zenity和dialog不太一样，它许多控件类型都用命令行上其他选项定义，而不是将它们包括在选项的参数中，具体的控件和它需要的参数选项可以在help中查看。

```shell
zenity --list --radiolist --title "SYs Admin Menu" --column "Select" --column "MenuItem" FALSE "Display disk space" FALSE "Display users" FALSE "Display memory usage" FALSE "Exit program" >$temp2  #列表框
zenity --info --text "Sorry,invalid selection"  #提示框
zenity --text-info --title "Disk space" --filename=$temp --width 750 --height 10 #文本框
```

Zenity的默认输出是stdout，和dialog不同。
