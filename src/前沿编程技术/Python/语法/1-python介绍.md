
## 几个小问题

- 选择python3还是python2？
别犹豫了，都什么年代了，直接选python3。Python的设计者觉得python前两个版本的设计太过零散，而且和其他编程语言比起来，有些地方不合理，导致欠他语言迁移过来的人难以适应，所以重新设计了python3。试问，设计者都在鼓励使用新版本，为何初学者守着旧版本不放（这又不是windows！)。除非你要使用的库确实没有python3版本，我都建议你从python3学起，其他语言迁移过来的开发者更应如此。
- 开发环境
认真告诉你，只推荐两个：Pycharm和visual studio code。前者是IDE（集成开发环境），最为推荐，智能提示、代码风格统统拥有，像一位老专家指导你写python。当然，如果你的PC比较差，跑不动Pycharm，就用visual studio code吧，自己配置上插件，也不差。Sublime和Atom我就不推荐了，一个收费，一个性能太差。远古的两位大神级vim和emacs更不推荐，学python不要折腾编辑器了（虽然我工作一直在用，一直在折腾）。_另外jupyter notebook作为学习时练习的工具很值得推荐，但是正规的开发工作，有点捉衿见肘了。_

---


## python包管理工具pip

如果你用过linux，一定对linux的软件包管理印象深刻： 安装一个软件，只需要在命令行敲击相应的命令，软件包就可以自定下载安装，不需要跑到网页上下载下来在安装。类似与linux上的apt/yum等工具，python也自带了包管理工具，可以让我们方便的安装和卸载python的包和库：


### pip源设置

pip也需要设置包的源（和linux一样的啦），默认使用官方源，不过一般比较慢。源设置很简单，在个人home目录打开.pip/pip.conf(linux平台)或个人user目录打开pip/pip.ini(windows平台)，添加如下内容：

```
[global]
index-url = http://pypi.douban.com/simple
[install]
trusted-host=pypi.douban.com
```

国内比较常见的pip源有如下几个，可以安装上面的格式自行修改：

- 阿里云 [http://mirrors.aliyun.com/pypi/simple/](http://mirrors.aliyun.com/pypi/simple/)
- 中国科技大学 [https://pypi.mirrors.ustc.edu.cn/simple/ ](https://pypi.mirrors.ustc.edu.cn/simple/%20)
- 豆瓣(douban) [http://pypi.douban.com/simple/](http://pypi.douban.com/simple/)
- 清华大学 [https://pypi.tuna.tsinghua.edu.cn/simple/](https://pypi.tuna.tsinghua.edu.cn/simple/)
- 中国科学技术大学 [http://pypi.mirrors.ustc.edu.cn/simple/](http://pypi.mirrors.ustc.edu.cn/simple/)


### pip命令详解

1. 安装包

```
# pip install SomePackage
  [...]
  Successfully installed SomePackage
```

2. 查看已安装的包

```
# pip show --files SomePackage
  Name: SomePackage
  Version: 1.0
  Location: /my/env/lib/pythonx.x/site-packages
  Files:
   ../somepackage/__init__.py
   [...]
```

3. 检查需要更新的包

```
# pip list --outdated
  SomePackage (Current: 1.0 Latest: 2.0
```

4. 升级包

```
# pip install --upgrade SomePackage
  [...]
  Found existing installation: SomePackage 1.0
  Uninstalling SomePackage:
    Successfully uninstalled SomePackage
  Running setup.py install for SomePackage
  Successfully installed SomePackage
```

批量升级包：

```python
from pip._internal.utils.misc import get_installed_distributions
from subprocess import call

for dist in get_installed_distributions():
    call("pip install --upgrade " + dist.project_name, shell=True)
```

5. 卸载包

```
pip uninstall SomePackage
  Uninstalling SomePackage:
    /my/env/lib/pythonx.x/site-packages/somepackage
  Proceed (y/n)? y
  Successfully uninstalled SomePackage
```

---

