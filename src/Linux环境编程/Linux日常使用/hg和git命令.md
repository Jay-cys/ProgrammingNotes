git强行push，覆盖服务器版本：

> git push -u origin master -f


# git常用命令：
![git](.assets/111111.jpg)

---


# hg和git命令对比
| 比较项目 | Hg命令 | Git命令 |
| --- | --- | --- |
| URL | [http://host/path/to/repos](http://host/path/to/repos) | git://host/path/to/repos.git |
|  | [ssh://user@host/path/to/repos]() | [ssh://user@host/path/to/repos.git]() |
|  | [file:///path/to/repos](file://path/to/repos) | [user@host](mailto:user%40host):path/to/repos.git |
|  | /path/to/repos | [file:///path/to/repos.git](file://path/to/repos.git) |
|  |  | /path/to/repos.git |
| 配置 | [ui]      username = Firstname Lastname [mail@addr](mailto:mail@addr) | [user]          name = Firstname Lastname          email = mail[@addr ](/addr )  |
| 版本库初始化 | hg   init  | git init [–bare]  |
| 版本库克隆 | hg   clone   | git   clone   |
| 获取版本库更新 | hg pull –update | git   pull |
| 更新至历史版本 | hg   update -r  | git   checkout  |
| 更新到指定日期 | hg   update -d  | git checkout HEAD@’{}’ |
| 更新至最新提交 | hg   update | git   checkout master |
| 切换至里程碑 | hg   update -r  | git   checkout  |
| 切换至分支 | hg   update -r  | git   checkout  |
| 还原文件/强制覆盖 | hg   update -C  | git checkout –  |
| 添加文件 | hg   add  | git   add  |
| 删除文件 | hg   rm  | git   rm  |
| 添加及删除文件 | hg   addremove | git   add -A |
| 移动文件 | hg   mv   | git   mv   |
| 撤消添加、删除等操作 | hg   revert  | git reset –  |
| 清除未跟踪文件 | hg   clean | git   clean -fd |
| 获取文件历史版本 | hg   cat -r  >  | git   show : >  |
| 反删除文件 | hg   add  | git   add  |
| 工作区差异比较 | hg   diff | git   diff |
|  |  | git diff –cached |
|  |  | git   diff HEAD |
| 版本间差异比较 | hg   diff -r  -r   | git   diff   –  |
| 查看工作区状态 | hg   status | git   status -s |
| 提交 | hg commit -m “” | git commit -a -m “” |
| 推送提交 | hg   push | git   push |
| 显示提交日志 | hg   log  less | git   log |
|  | hg   glog  less | git log –graph |
| 逐行追溯 | hg   annotate | git   annotate, git blame |
| 显示里程碑/分支 | hg   tags | git   tag |
|  | hg   branches | git   branch |
|  | hg   heads | git   show-ref |
| 创建里程碑 | hg tag [-m “”] [-r ]  | git tag [-m “”]  [] |
| 删除里程碑 | hg tag –remove  | git   tag -d  |
| 创建分支 | hg   branch  | git   branch   |
|  |  | git   checkout -b   |
| 删除分支 | hg commit –close-branch | git   branch -d  |
| 导出项目文件 | hg   archive -r  <output.tar.gz> | git   archive -o <output.tar>  |
|  |  | git   archive -o <output.tar> –remote=  |
| 反转提交 | hg   backout  | git   revert  |
| 提交拣选 | - | git   cherry-pick  |
| 分支合并 | hg   merge  | git   merge  |
| 变基 | hg   rebase | git   rebase |
| 冲突解决 | hg resolve –tool= | git   mergetool |
|  | hg   resolve -m  | git   add  |
| 更改提交说明 | Hg   + MQ | git commit –amend |
| 撤消最后一次提交 | hg   rollback | git reset [ –soft \\ –hard ] HEAD^ |
| 撤消多次提交 | Hg   + MQ | git reset [ –soft \\ –hard ] HEAD~ |
| 撤消历史提交 | Hg   + MQ | git   rebase -i ^ |
| 启动Web浏览 | hg   serve | git   instaweb |
| 二分查找 | hg   bisect | git   bisect |
| 内容搜索 | hg   grep | git   grep |
| 提交导出补丁文件 | hg   export | git   format-patch |
| 工作区根目录 | hg   root | git rev-parse –show-toplevel |
| 杂项 | .hgignore 文件 | .gitignore 文件 |
|  | pager 扩展 | 内置分页器 |
|  | color 扩展 | color.* 配置变量 |
|  | mq 扩展 | StGit,   Topgit |
|  | graphlog 扩展 | git log –graph |
|  | hgk 扩展 | gitk |

