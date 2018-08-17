---
title: Getting Started with Git
date: 2018-08-15 14:50:13
tags:
---

{% asset_img header.jpg 500 250 %}
> Git是一个分布式的版本控制软件，不需要服务器端软件，就可以运作版本控制，使得代码的发布和交流机器方便。Git的速度很快，这对于Linux内核的项目来说很重要，且Git拥有出色的合并追踪(merge tracking)能力，能够记录修改的代码。
>


<!--more -->

# 安装配置Git
Linux下安装Git非常的简单，只需要简单的一条命令。
```bash
sudo apt-get install git
```
配置本地Git的用户名和邮箱与Github的一致
```bash
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

# Git基本概念

## What is git?
git是一个管理代码的版本控制系统(Version Control System, VCS)，它能跟踪每一个文件的变化，如果你修改了某个文件，VCS能够记录并且保存这些变化，这使得你可以撤销任何的修改，回溯到任何一个历史版本。
**注意**
所有的版本控制系统都只能跟踪文本文件的改动，如TXT文件、网页文件、程序代码等；而对于图片、视频这种二进制文件，虽然也能由版本控制系统管理，但没法跟踪文件的变化，只能把二进制文件每次改动串起来，也就是只能知道图片从100KB改成120KB，但是到底改了什么，无法知道。不幸的是，Microsoft的Word文件也是二进制格式的，因此Git也没法跟踪Word文件的改动。

## Git workflow
在一个Git项目里面，由三个主要的组成元素：
- 版本库(repository)：简称repo，它记录了你项目的所有变化和修改，保存了所有的commit，commit是指在某个时间点，项目所有文件的一个snapshot，可以理解为一次提交，将最新的版本提交到版本库，交由版本库进行保存。
- 暂存区(index or staging area)：它是工作区和版本库之间的一个bridge，可以理解为一个中转站。暂存区的文件能够被commit，index is where commits are prepared。
- 工作区(working tree)：就是我们本地工作的目录，我们能够修改文件、增添删除文件的目录。

三者可以理解为一个层级关系，版本库是在层级最高的位置，工作区最低，我们在工作区工作，进行文件修改，我们每完成一次修改，我们希望保存到版本库，并且告诉版本库这次修改了什么，好让版本库能够记录下来。
下面就是Git的基本工作流程：
1. 在工作区修改文件。
2. 将工作区所有修改过的文件add(stage)到暂存区，准备commit。
3. commit到版本库。
{% asset_img git_workflow.png 800 400 %}

一个文件的四个状态，可以通过git status命令看得到这些词。
- Untracked: 当一个文件新创建，在版本库里面没有它，所以无法追踪。
- Modified：一个文件在工作区被修改了，但是还没到暂存区。
- Staged：修改的文件被stage到暂存区。
- Committed：修改的文件在暂存区commit到版本库。
{% asset_img git_four_states.png 800 400 %}

Git项目的三个组成元素，各自代表一个版本，我们可以查看两两之间的差别：
- git diff: 查看工作区和暂存区的差别。
- git diff --cached: 查看暂存区和版本库的差别。
- git diff HEAD: 查看工作区和版本库的差别。


## 版本回退
### 仓库版本回退
Git既然能够跟踪文件的修改，自然就可以回溯，`.git`文件有保留每一次commit的信息，每一次commit都有一个唯一的commit id对应，它是经过SHA1计算出来的一个数字，用16进制表示，我们可以通过命令回退命令，回退到想要版本。
```
git reset <mode> <你想回退到的版本的commit id的前几位>
```

在Git管理版本的时候，会有一个HEAD指针，指向最近的一次commit，每次的commit会练成一条链，回退的过程实际是将HEAD指针指向之前的节点
{% asset_img git_reset.png 400 200 %}

reset有三种mode：soft, mixed, hard，可以看到无论是哪个模式，HEAD的位置都肯定是会改变的，所以reset的另外一个作用就是撤销commit。

| mode  | HEAD指针的位置 | 暂存区 | 工作区 |
| :---: | :------------: | :----: | :----: |
| soft  |      改变      |  不变  |  不变  |
| mixed |      改变      |  改变  |  不变  |
| hard  |      改变      |  改变  |  改变  |

一种更加简单的方式替换commit id的是用HEAD指针，上一个版本就是`HEAD^`，上上个版本就是`HEAD^^`，依次往后可以写成`HEAD~n`。

### 工作区修改撤销
当你在工作区乱改一通，也忘了自己改了什么地方，那么你可以用本地版本库的内容直接替换工作区，丢弃工作区的所有改变，用以下的命令可以使得file回到最近一次commit或者stage时的状态(注意，stage也是可以的，有些人可能提交到暂存区后，又乱改一通，这条命令将暂存区的版本替换工作区)
```
git checkout -- <file>
```

### 暂存区版本回退
当我们在commit之前发现文件有问题，不能commit，但是修改的文件已经添加到了暂存区了，可以用以下的命令将暂存区的修改撤销(unstage)，重新放回工作区。`git reset`命令既可以回退版本，也可以把暂存区的修改回退到工作区，用HEAD的时候，表示最新的版本。
```
git reset HEAD <file>
```

总结一下：
1. 撤销工作区修改，可以直接用本地版本库来替换
2. 不小心stage到暂存区，可以用reset命令撤销，撤销后暂存区该文件恢复原状，但是工作区依旧时修改了的。但是我个人觉得，既然工作区你瞎改了一些东西，并且add到暂存区，那么只需要把修改的内容改回去，再重新add到暂存区，就能恢复原状了。
3. 不小心还commit到版本库了，也可以用reset命令，结合hard mode，将工作区、暂存区全部恢复之前的版本。

## 创建版本库
首先，版本库有两种类型：
- 本地版本库(local)，寄托在本地的机器，一般是个人电脑，为个人所用。
- 远端版本库(remote)，托管在远端的一个服务器上，被多个用户所使用。Github就是全球最大的托管Git版本库的服务器，从名字就可以知道。

本地版本库的创建有两种方法：
- git init, 在本地初始化一个版本库。
- git clone, 从远端克隆一个仓库到本地。

当我们创建完一个本地版本库后，在本地的目录下就会多一个`.git`的文件夹，它就是版本库，里面就是管理整个目录的内容。

## 版本库的同步
远端仓库的创建非常简单，只需要在Github上创建即可，但是远端仓库创建之后，仓库内是空的，它只能通过本地版本库的同步来实现初始化。
远端仓库使得我们可以和其它用户一起合作，只需要保证远端的版本库永远是最新的版本即可，因此，本地和远端版本库之间就需要频繁地进行同步，主要通过三个动作来完成：push, pull, merge。
同步之前，我们需要绑定本地版本库到远端仓库，才能进行以后的同步操作。远端仓库默认叫做origin，当然也可以叫其它名字。
```
git remote add origin <remote repo addr>
```

### Git push
当我们想将本地版本库更新到远端仓库的时候，我们可以将其push到远端的仓库，这样就能使得远端仓库跟本地保持同步。
{% asset_img push.png 500 250 %}

### Git pull
当你的队友将他的本地仓库push到你们共同的远端仓库，你需要对自己本地的仓库进行更新，跟你队友的仓库保持一致，这时候我们就需要将远端仓库pull到自己的本地仓库。
{% asset_img pull.png 500 250 %}

### Git merge
无论是上传到远端仓库还是从远端仓库下载，远端仓库永远是最新版本，拥有最高优先级的。当我们想push我们的本地仓库到远程仓库的时候，不幸你的本地库的版本不是最新版本(即你在修改自己的本地版本的同时，有人更新了远程仓库)，这时候push会被拒绝，因为远程仓库的有些更新并不在你的本地仓库。这时候我们需要将远端仓库先pull下来，并和本地仓库进行合并(merge)。
merge就是指两个仓库(或者一个仓库的两个分支)进行合并的过程，在合并的过程，Git会自动地将另外一个仓库(分支)的改变更新到当前的仓库(分支)。
但是，合并的两个仓库(分支)在某些文件可能都共同地修改了同个地方，这时候就出现了所谓的conflit，Git并不会自动地帮你选择一个版本，而是将这个选择的权利交给了我们，Git会帮我们将conflit的地方标记出来，我们需要手动地进行修改，才能最后完成合并过程。

下面对比一下pull和fetch
- fetch：fetch是将远程仓库的某条分支的内容拉到本地，但是fetch后是看不到变化，而是在本地新开了一个分支，该分支的指针是`FETCH_HEAD`，checkout到该分支后可以查看远程分支的最新内容。然后切换到master分支，执行merge，选中`FETCH_HEAD`，合并后如果出现冲突则解决冲突，最后commit。
- pull：pull相当于fetch和merge，自动将远程仓库更新到本地仓库
```
git fetch origin master(将远程仓库的master分支拉到本地当前分支)
git merge FETCH_HEAD
```

# 分支管理和协作
分支使得Git变得更加的强大，使得团队协作更加的方便。任何一个Git仓库，都会默认有一个master的分支，无论是本地仓库还是远程仓库，这个分支是在创建Git仓库的时候就会默认创建的。之前提到HEAD指针是指向最近一次commit的，但严格来讲，HEAD指针是指向master指针，master指针才是指向最近一次commit，所以可以理解为HEAD指针是指向当前的分支。
如果这样来看的话，一条分支就好比链表，每一次commit就往链表尾部插入新的仓库snapshot。一个仓库可以有好几条分支，当新开一条分支的时候，原有的链表尾部就会开始分叉，同时会有另外一个新的指针指向新的分叉，当我们切换分支的时候，HEAD指针就会指向对应分支的指针。
{% asset_img branch.png 500 250 %}

## 创建、切换分支
Git创建分支和切换分支都很快，因为无非就是创建一个指针，和改变HEAD指针的指向而已。
```
创建branch：git branch <branch_name>
切换branch：git checkout <branch_name>
创建并切换branch：git checkout -b <branch_name>
查看branch list：git branch
```

## 分支合并
在多人协作的时候，基本上都是每个人都在自己的分支上工作，完成自己的工作之后再把自己的代码合并到master分支上。
之前提到，两条分支的merge可能会导致conflit，冲突的原因是两个**已经提交的分支**的相同文件相同位置的不同操作进行了合并。

**注意**
要避免冲突，就是最好每次修改文件之前，先merge别的分支(或者pull远程仓库)，这样就能保证自己是在别人最新版本的基础上修改的，自己修改完后去合并到别人分支(push到远程仓库)都不会产生冲突。
这种情况就好比下图，第三个节点是最新版本，然后在dev分支上修改，修改完成后commit，再切换回master分支，然后跟dev分支进行merge，这时候这种merge叫做fast forward，因为是直接将master的指针指向了第四个节点，相当于直接覆盖。
{% asset_img ff_merge.png 500 250 %}

以下的情况会导致冲突，假如我在dev分支修改了a文件的第二行代码，并且提交了。然后我切换到master分支，假如我不知道dev分支改了什么，我在master分支也改了a文件的第二行代码，并且提交了。然后我在这个时候想把dev分支的修改一起merge到master分支上，冲突发生了，因为两个版本都修改了同一行代码，Git会将冲突的位置，用以下的方式告知我们，并要求我们人工进行修改。如果不解决冲突时没法提交或者切换分支的。
```
<<<<<<<< HEAD(Current Change)
other code
========
your code
>>>>>>>> your branch name(Incoming Change)
```

但是假如我在master分支改的时a文件的第三行代码，并且提交了，这时候再去将dev分支merge进来，不会有冲突，而是会自动合并，即使我在修改第三行代码前并没有先将dev的修改merge进来。但是还是建议先merge再做修改。

## 与远端仓库的同步
像之前所说，无论是本地仓库还是远端的仓库，都可以存在不同的分支，还是那句话，在修改代码前，先将远端仓库pull下来，方便以后push的时候，避免产生冲突。并且需要注意，本地仓库是从哪条分支pull下来的，最好就push回哪条分支，不然push不上去。
如果本地仓库和远端仓库都只有一条分支，那么情况就简单很多，因为不需要明确地指明哪条分支到哪条分支。
- pull：如果本地和远端都只有一条分支，直接git pull就好，如果想pull到当前分支，那么本地分支名可以省略。
    ```
    git pull [远程仓库名字，一般默认origin] [远端分支名]:[本地分支名]
    eg: git pull origin master:master
    ```
- fetch：fetch不需要写本地分支名，因为它还没有merge，只是把远端分支拉到本地并保存到`FETCH_HEAD`而已。
    ```
    git fetch [远程仓库名字] [远端分支名]
    ```
- push：建议写全。
    - 最常见到的是`git push origin master`，远程分支被省略，它表示将本地分支推送到与其存在追踪关系的远程分支(通常两者同名)，因为远端仓库肯定存在master分支，因此省略也没有问题。如果该远程分支不存在，则会被新建。
    - 其它形式如`git push origin`，表示将当前分支push到远端与当前分支存在追踪关系的分支。
    - `git push`，如果本地和远端都只有一条分支，那么全都可以省略。
    ```
    git push [远程仓库名字] [本地分支名]:[远端分支名]
    ```

## Stash
Stash是一个工作状态保存栈，用于保存/恢复工作区的临时状态。
{% asset_img stash.png 500 250 %}

那么什么时候才需要将工作区的状态暂时保存起来呢？说到暂时，那么肯定就是修改只进行了一半，还没到commit或者stage的地步。比如我在master上进行了一些修改，但是还没有commit(加到暂存区也不行)，现在我需要切换到dev分支进行其它的修改。Git会reject你的分支切换(另一种reject分支切换的情况是出现conflit，conflit没解决之前，不允许切换分支)，并且告诉你要不将修改commit，要不将它放到stash里面，才可以切换分支。这就是stash出现的目的，暂时存储工作区的状态。
将工作区临时保存起来可以用`git stash`命令，实际保存的是工作区的一个snapshot，将工作区stash之后，工作区变回干净状态(从git status可以看出)。因此可以多次stash，相当于将不同的几个snapshot保存起来，stash的地方是一个栈，遵循后进先出。
`git stash list`可以查看栈里面的snapshot。如果要恢复工作区可以有两种方法：
- `git stash pop`：将栈顶元素pop出来，恢复工作区的同时把stash的内容删掉。
- `git stash apply stash@{n}` & `git stash drop stash@{n}`：从stash list选出需要恢复的snapshot，snapshot的命名就是`stash@{n}`。


# 总结
Git可以说是每一个开发者必备的工具，而Github更是全世界最活跃的网站之一，无论你是将Github看成是一个项目代码的仓库，还是在公司跟同事合作，掌握Git都会让你受益匪浅，个人推荐用Git Bash，不要依赖Github Desktop，虽然方便，但是沉下心来理解Git的基本概念和操作，也是每个开发者值得做和应该做的一件事。

# 参考资料
1. [廖雪峰Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
2. [backlog git tutorial](https://backlog.com/git-tutorial/)
3. [Git官网](https://git-scm.com/docs/)，但是太难懂了，我暂时也没看懂>_<