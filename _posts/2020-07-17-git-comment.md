\---

layout: post

title:Git常用操作总结

categories:front-end

description: Git常用操作总结

keywords: git

\---

在进入公司实习之前，一直用的都是GitHub desktop可视化工具，从来没用过git命令，然后实习以后发现公司并不用GitHub，所以被迫学起git的常用命令，两个星期以来也总结了一下几个常用的命令：

`git config -l`查看git配置信息

`git checkout-b <branch>`从当前分支创建并转入新分支

`git status`查看当前分支工作区和暂存区的状态

`git diff`diff文件的修改（很重要！）

`git log`查看提交记录

`git branch`查看本地所有分支

`git branch -a`查看远程和本地所有分支

`git fetch --p`更新分支

`git checkout -- <filePathName>` 放弃单个文件修改（未使用 git add 缓存代码）

`git checkout .`放弃所有的文件修改(未使用 git add 缓存代码)

`git checkout <branch>`转入已存在的分支

`git reset HEAD <filePathName>`如果已经使用了 git add 缓存了代码,可以用来放弃指定文件的缓存

`git reset --hard <commitId>` 如果已经用 git commit 提交了代码，此命令可以用来回退到任意版本

`git fetch --all`拉取所有远程最新代码

`git remote` 不带参数，列出已经存在的远程分支

`git remote -v`检查是否关联成功

`git add .`缓存所有追踪的更改文件

`git clean -f`删除当前目录下所有没有track过的文件

`git commit -m ""`提交缓存区的代码

`git push origin <branch>`上传提交至指定分支

`git merge <branch>`合并分支

*可能随实际开发会碰到更多的情况，会及时更新*