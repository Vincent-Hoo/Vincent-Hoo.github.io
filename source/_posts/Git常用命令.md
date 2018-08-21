---
title: Git常用命令
date: 2018-08-18 00:15:28
tags:
categories:
- Git
---

{% asset_img header.jpg 500 250 %}
该文章总结了最常用的Git命令，包括创建仓库、更新仓库、与远端仓库同步等。
参考了文件 {% asset_link git-cheatsheet.pdf %}，并在此基础上进行了一定的补充。

<!-- more -->

- 创建仓库
```
$ git init
```

- 从远端克隆仓库
```
$ git clone <repo addr>
```

- 查看工作区和暂存区的状态。该命令会返回相比于暂存区，工作区的状态改变信息，如new file，modified，deleted file等
```
$ git status
```

- 将工作区的代码添加到暂存区，为commit做准备
```
$ git add [参数] [路径]
$ git add . : 将修改的文件和新添加的文件stage到暂存区，不包括删除的文件
$ git add --all(-A)：将所有已跟踪的文件的修改与删除和新添加的未跟踪的文件添加到暂存区
```

- 将暂存区的代码，提交到本地版本库。-m 参数表示可以直接输入后面的“message”，如果不加 -m参数，那么是不能直接输入message的，而是会调用一个编辑器一般是vim来让你输入这个message
```
$ git commit -m <commit message>
```

- 追加commit，这是直接在上一次的commit基础上进行modify，而不会新开一个commit id，该命令比较少用
```
$ git commit --amend
```

- 查看不同区域状态的差异
```
$ git diff: 查看工作区和暂存区的差别
$ git diff --cached: 查看暂存区和版本库的差别
$ git diff HEAD: 查看工作区和版本库的差别
```

- 查看历史的commit记录，可以添加参数，比如缩写(正常的commit id很长)，还可以让它一条commit显示一行
```
$ git log (--pretty=oneline) --abbrev
```

- 查看所有分支的commit记录，包括commit，reset等，回退后的commit在git log是看不到的，而git reflog可以
```
$ git reflog
```

- 查看当前仓库的分支，参数`-r`表示查看远端仓库的分支
```
$ git branch
$ git branch -r
```

- 新建一条分支
```
$ git branch <branch name>
```

- 切换分支
```
$ git checkout <branch name>
```

- 新建并切换分支
```
$ git checkout -b <branch name>
```

- 删除分支，参数-d还是-D，取决于这条分支的改变有没有merge到其它分支
```
$ git checkout -d(D) <branch name>
```

- 撤销commit
```
$ git revert <刚刚提交的commit id>
```

- 版本回退，mode一般都会选择hard
```
$ git reset <mode> <commit id或者HEAD指针倒退>
```

- 撤销工作区某个文件的修改
```
$ git checkout -- <file name>
```

- 撤销stage
```
$ git reset HEAD <file>
```

- 合并分支，默认是--ff，fast forward，也可以选择--no-ff，那就会新添加一个commit，名为merged
```
$ git merge <mode> <branch name>
```

- 绑定远端仓库，仓库名一般就叫origin就ok
```
$ git add remote <远端仓库名> <仓库地址>
```

- 更新远端仓库
```
$ git push <远端仓库名> <本地分支名>:<远端分支名>
```

- 更新本地仓库
```
$ git pull <远端仓库名> <远端分支名>:<本地分支名>
```

- 将远端仓库拉到本地
```
$ git fetch <远端仓库名> <远端分支名>
```

- 将工作区暂时保存
```
$ git stash
```

- 恢复工作区
```
$ git stash pop
$ git stash apply stash@{n} + git stash drop stash@{n}
```

<br>