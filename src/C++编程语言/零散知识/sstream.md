在解决leetcode[题目8问题](https://leetcode-cn.com/problems/string-to-integer-atoi/)（从字符串中取出连续的数字子字符串并转换为数字）时，了解到一种比顺序解析和正则表达式更快代码量更少的方法，代码如下，使用到了istringstream类。

```cpp
#include <sstream>
int myAtoi(string str) 
{
    int d = 0;
	istringstream is(str);
	is >> d;
	return d;
}
```

所以对该类多增加一些了解：


## 头文件

`#include <sstream>`

sstream定义了三个类：**istringstream**、**ostringstream** 和 **stringstream**，分别用来进行流的输入、输出和输入输出操作。 sstream主要用来进行数据类型转换，相比c库的数据类型转换而言，sstream更加安全、自动和直接，而且传入参数和目标对象的类型会被自动推导出来。


## istringstream

istringstream类用于执行**C++风格的串流的输入操作**, istringstream对象可以绑定一行字符串，然后**以空格为分隔符**把该行分隔开来，将输入分别对应到要保存的变量中。

leetcode题目8的代码中，要求转换代码空格之后出现的首个数字子字符串，且如果空格后不为正负号或数字，就返回0，正好对应istringstream的用法。我们希望从输入流中得到的第一个输入为一个INT类型，刚好可以符合题目要求，而且如果数字超过INT范围，istringsteam会自动做溢出操作。


## ostringstream

ostringstream类通常用于执行**C风格的串流的输出操作**，格式化字符串，避免申请大量的缓冲区，替代sprintf。

**NOTE**: ostringstream::str()返回的是临时对象，不能对其直接操作。

```cpp
#include <sstream>  
#include <string>  
#include <iostream>  
using namespace std;  
  
void main()  
{  
    ostringstream ostr1; // 构造方式1  
    ostringstream ostr2("abc"); // 构造方式2  
  
/*---------------------------------------------------------------------------- 
*** 方法str()将缓冲区的内容复制到一个string对象中，并返回 
----------------------------------------------------------------------------*/  
    ostr1 << "ostr1 " << 2012 << endl; // 格式化，此处endl也将格式化进ostr1中  
    cout << ostr1.str();   
  
/*---------------------------------------------------------------------------- 
*** 建议：在用put()方法时，先查看当前put pointer的值，防止误写 
----------------------------------------------------------------------------*/  
    long curPos = ostr2.tellp(); //返回当前插入的索引位置(即put pointer的值)，从0开始   
    cout << "curPos = " << curPos << endl;  
  
    ostr2.seekp(2); // 手动设置put pointer的值  
    ostr2.put('g');     // 在put pointer的位置上写入'g'，并将put pointer指向下一个字符位置  
    cout << ostr2.str() << endl;  
      
  
/*---------------------------------------------------------------------------- 
*** 重复使用同一个ostringstream对象时，建议： 
*** 1：调用clear()清除当前错误控制状态，其原型为 void clear (iostate state=goodbit); 
*** 2：调用str("")将缓冲区清零，清除脏数据 
----------------------------------------------------------------------------*/  
    ostr2.clear();  
    ostr2.str("");  
  
    cout << ostr2.str() << endl;  
    ostr2.str("_def");  
    cout << ostr2.str() << endl;  
    ostr2 << "gggghh";    // 覆盖原有的数据，并自动增加缓冲区  
    cout << ostr2.str() << endl;
    ostr2.str("");   // 若不加这句则运行时错误，因为_df所用空间小于gggghh，导致读取脏数据
    ostr2.str("_df");  
    cout << ostr2.str() << endl;
 
    // 输出随机内存值，危险
    const char* buf = ostr2.str().c_str();  
    cout << buf << endl;
 
    // 正确输出_df
    string ss = ostr2.str();
    const char *buffer = ss.c_str();
    cout << buffer << endl;
}
```


## stringstream

stringstream是双向的，**一般用来进行类型转换**，主要用途如下：


### 1 类型之间转换

比如，将int转换为string：

```cpp
stringstream sstream;
string strResult;
int nValue = 1000; 
// 将int类型的值放入输入流中
sstream << nValue;
// 从sstream中抽取前面插入的int类型的值，赋给string类型
sstream >> strResult;
```


### 2 多个字符串拼接

在 stringstream 中存放多个字符串，实现多个字符串拼接:

```cpp
stringstream sstream;
// 将多个字符串放入 sstream 中
sstream << "first" << " " << "string,";
sstream << " second string";
cout << "strResult is: " << sstream.str() << endl;//通过str函数一次性全部取出
 
// 清空 sstream
sstream.str("");
```


### stringstream的清空

上面可以使用str("")来清空，其次还可以使用clear函数清空， **clear() 方法适用于进行多次数据类型转换的场景**（因为不一定是string类型，所以写入""是不合适的）。

```cpp
stringstream sstream;
int first, second;
 
// 插入字符串
sstream << "456";
// 转换为int类型
sstream >> first;
cout << first << endl;
 
// 在进行多次类型转换前，必须先运行clear()
sstream.clear();
 
// 插入bool值
sstream << true;
// 转换为int类型
sstream >> second;
cout << second << endl;
```
