
## 1. 为什么使用日志打印

一般人喜欢用print打印一些变量或者执行路径，以此来作为程序的调试手段，但不要屈服于这种诱惑！以实际工作经验来看，非常的不可取，我来列举几点原因：

1. 正常程序运行时，不应该打印与程序无关的内容，只能打印用户想看到的内容。用户不会运行程序在正常执行时，屏幕上出现一些调试信息。
2. 在调试完成后，你需要花很多时间，从代码中清除每条日志消息的 print() 调用。你甚至有可能不小心删除一些 print() 调用，而它们不是用来产生日志消息的。相信我，项目很大时，你添加很多print打印，要把这些print删除，不是一件简单的事。一个文件尚且删错，一万个呢？
3. 不要指望断点调试。比如我所在的项目代码是运行在一块独立的板卡上的，里面只有代码在运行，根本没有调试环境让你打断点；而95%的功能只有在板卡上才能运行，PC上无法模拟。如何使用断点？

因此，老老实实使用日志打印吧。日志打印使log的显示和隐藏格外容易。出bug的时候打开日志模块，打印程序执行时候的log；没有bug时，关闭日志模块，没有多于打印，也不会影响性能（这个准则不区分编程语言的）。
在Python中，logging 模块使你很容易创建自定义的消息记录。这些日志消息将描述程序执行何时到达日志函数调用，并列出你指定的任何变量当时的值。另一方面，缺失日志信息表明有一部分代码被跳过，从未执行。只要加入一次logging.disable （logging.CRITICAL）调用，就可以禁止日志。


## 2. logging模块

Example：

```python
import logging
#基本配置
logging.basicConfig(level=logging.DEBUG, format=' %(asctime)s - %(levelname)s -%(message)s')
#打印debug级别log
logging.debug('Start the program')

def factorial(n):
    logging.debug('Enter function factorail with input {0}'.format(n))
    total = 1
    for i in range(n+1):
        total*=i
        logging.debug('i is {0}, total is {1}'.format(i, total))

    logging.debug('Exit function factorial')
    return total

print(factorial(5))
logging.debug('End the program')
```


### 2.1 basicConfig参数

- filename: 指定保存日志文件名
- filemode: 和file函数意义相同，指定日志文件的打开模式，'w'或'a'
- format: 指定输出的格式和内容，format可以输出很多有用信息，如上例所示:

   - %(levelno)s: 打印日志级别的数值
   - %(levelname)s: 打印日志级别名称
   - %(pathname)s: 打印当前执行程序的路径，其实就是sys.argv[0]
   - %(filename)s: 打印当前执行程序名
   - %(funcName)s: 打印日志的当前函数
   - %(lineno)d: 打印日志的当前行号
   - %(asctime)s: 打印日志的时间
   - %(thread)d: 打印线程ID
   - %(threadName)s: 打印线程名称
   - %(process)d: 打印进程ID
   - %(message)s: 打印日志信息
- datefmt: 指定时间格式，同time.strftime()
- level: 设置日志级别，默认为logging.WARNING
- stream: 指定将日志的输出流，可以指定输出到sys.stderr,sys.stdout或者文件，默认输出到sys.stderr，当stream和filename同时指定时，stream被忽略


### 2.2 log级别
| 级别 | 日志函数 | 描述 |
| --- | --- | --- |
| DEBUG | logging.debug() | 最低级别。用于小细节。通常只有在诊断问题时，你才会关心这些消息 |
| INFO | logging.info() | 用于记录程序中一般事件的信息，或确认一切工作正常 |
| WARNING | logging.warning() | 用于表示可能的问题，它不会阻止程序的工作，但将来可能会 |
| ERROR | logging.error() | 用于记录错误，它导致程序做某事失败 |
| CRITICAL | logging.critical() | 最高级别。用于表示致命的错误,它导致或将要导致程序完全停止工作 |



### 2.3 禁用日志

在调试完程序后，你可能不希望所有这些日志消息出现在屏幕上。 **logging.disable()** 函数禁用了这些消息，这样就不必进入到程序中，手工删除所有的日志调用。只要向 logging.disable() 传入一个**日志级别**，它就会禁止该级别和更低级别的所有日志消息。

因为 logging.disable() 将禁用它之后的所有消息，你可能希望将它添加到程序中接近 import logging 代码行的位置。这样就很容易找到它，根据需要注释掉它，或取消注释，从而启用或禁用日志消息。
