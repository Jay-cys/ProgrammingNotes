
## 1、简介

MD5算法常常被用来验证网络文件传输的完整性，防止文件被人篡改。MD5 全称是报文摘要算法（Message-Digest  Algorithm  5），此算法对任意长度的信息逐位进行计算，产生一个二进制长度为128位（十六进制长度就是32位）的“指纹”（或称“报文摘要”），不同的文件产生相同的报文摘要的可能性是非常非常之小的。
在linux或Unix上，md5sum是用来计算和校验文件报文摘要的工具程序。一般来说，安装了Linux后，就会有md5sum这个工具，直接在命令行终端直接运行。


## 2、命令格式

md5sum [OPTION]... [FILE]...


## 3、命令选项

- -b 或 --binary :把输入文件作为二进制文件看待。
- -t 或 --text :把输入的文件作为文本文件看待（默认）。
- -c 或 --check :用来从文件中读取md5信息检查文件的一致性。(不细说了参见info)
- --status :这个选项和check一起使用,在check的时候，不输出，而是根据返回值表示检查结果。
- -w 或 --warn :在check的时候，检查输入的md5信息又没有非法的行，如果有则输出相应信息。


## 4、例子


### （1）生产一个文件的md5值，如下所示：

[root[@linuxidc ](/linuxidc ) www.linuxidc.com]# md5sum p182  > cc.txt 

这样，就生产了文件p182的md5校验文件cc.txt,打开cc.txt可以看到如下内容：
[root[@linuxidc ](/linuxidc ) www.linuxidc.com]# cat cc.txt 
dd05e55e3aca2d39e49a1a7d1b3fd404  p182


### （2）检查两个文件是否一样，可以通过比较两个文件的md5值

[root[@linuxidc ](/linuxidc ) www.linuxidc.com]# md5sum p182 > ee.txt 
[root[@linuxidc ](/linuxidc ) www.linuxidc.com]# cat ee.txt 
0f0b000b096ac947d6067466cc60ffd0  p182

[root[@linuxidc ](/linuxidc ) www.linuxidc.com]# md5sum p183 > ff.txt 
[root[@linuxidc ](/linuxidc ) www.linuxidc.com]# cat ff.txt 
dd05e55e3aca2d39e49a1a7d1b3fd404  p183


### （3）判断一个文件是否修改，通过md5来判断

[root[@linuxidc ](/linuxidc ) www.linuxidc.com]# md5sum p182 > cc.txt 
[root[@linuxidc ](/linuxidc ) www.linuxidc.com]# cat cc.txt 
0f0b000b096ac947d6067466cc60ffd0  p182

修改下p182文件：
[root[@linuxidc ](/linuxidc ) www.linuxidc.com]# vi p182 
跟原来的md5结果进行对比：
[root[@linuxidc ](/linuxidc ) www.linuxidc.com]# md5sum p182 -c cc.txt 
md5sum: p182: no properly formatted MD5 checksum lines found
p182: FAILED
md5sum: WARNING: 1 of 1 computed checksum did NOT match

修改后md5结果如下：
[root[@linuxidc ](/linuxidc ) www.linuxidc.com]# md5sum p182 > dd.txt 
[root[@linuxidc ](/linuxidc ) www.linuxidc.com]# cat dd.txt 
521e87e3331e94a483aec6bf19039845  p182


## 5、windows下查看MD5

有时候因为某些原因我们需要查看文件的MD5值，在Linux下这个就非常简单，只需要用md5sum命令即可，但是在Windows上却不知道对应的命令。今天就在网上查了一些，果然其实Windows也有对应的命令。而且该命令还可以查看SHA1值和SHA256值的功能。命令如下：

1. certutil -hashfile filename MD5
2. certutil -hashfile filename SHA1
3. certutil -hashfile filename SHA256
