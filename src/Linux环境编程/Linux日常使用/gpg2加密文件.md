
## 一、安装

按照官方页面的指导，下载 GnuPG，并安装。
在 Terminal 里测试安装是否成功：
`gpg2 --versioin`
注意，这里的命令是gpg2，不是gpg也不是pgp
检查/Users/buchiany/.gnupg目录权限所属用户/用户组，提前解决可能会出现的错误 Permission denied
使用命令ls -al查看子目录或文件权限
使用命令sudo chown -R buchiany:staff .gnupg/修改用户组为当前用户/用户组。


## 二、创建密钥对


### 1、创建

```
gpg2 --generate-key 按默认参数创建密钥对
            --full-generate-key 自定义参数创建密钥对
```

可选择密钥种类（默认是 RSA and RSA），密钥长度（默认2048），有效期限。
接下来会要求输入姓名、邮件地址和注释，来生成用户标识。英文，可只写姓名和邮件地址。
接下来会要求新建输入一个口令，这个口令是用来保护私钥。
成功生成密钥对之后，输出的结果格式类似如下：

```
/Users/username/.gnupg/pubring.kbx
---------------------------------
pub   rsa2048 2017-04-03 [SC]
      5880C75057FED0E3XXXXXXXXXXXXXXXXXXXXXXXX
uid           [ultimate] foobar <foo@bar.com>
sub   rsa2048 2017-04-03 [E]
```


### 2、输出并备份密钥

密钥是自己用来签名自己所发送的信息，或解密别人发来的加密信息（用此密钥配对的公钥加密）。
导出命令：`gpg2 -a -o gpg-private-key.txt --export-secret-keys`

- -a 以Ascii文本输出
- -o filename 输出到指定文件
换个地方保存好这个密钥备份，避免泄露。


### 3、输出公钥，告知他人

公钥是别人用来验证签名（不能签名），或加密（不能解密）要发送给你的信息。
导出命令：`gpg2 -a -o gpg-public-key.txt --export [用户ID]`

- "用户ID"是指定哪个用户的公钥
- -o filename 输出到指定的文件
然后就通过不同方式将公钥发给别人，比如挂在自己的网站上。
也上传公钥到专门的公钥服务器
命令：gpg2 --send-keys [用户ID] --keyserver hkp://subkeys.pgp.net
可以网上搜索有哪些公钥服务器


## 三、应用


### 加密信息

1）导入别人的公钥
2）加密文件

```
gpg2 -r 用户ID -o destfile -e sourcefile
-r 用户ID，指定用户ID即指定用来加密的公钥
-o destfile，"destfile"加密输出的文件
-e sourcefile，"sourcefile"是待加密的文件
```


### 解密信息

`gpg2 -o destfile -d sourcefile`
