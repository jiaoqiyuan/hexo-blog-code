---
title: Git使用笔记
date: 2019-03-17 13:11:12
tags: git
categories: linux
thumbnail: https://git-scm.com/images/branching-illustration@2x.png
photos: https://git-scm.com/images/branching-illustration@2x.png
---

> 最近在面试中碰到了Git使用的问题，自己答得并不好，这里总结一下git的使用方法，给自己也给需要的人看一下。

-------

<!--more-->

## Git的由来

Git是Linus开发的用于托管Linux源码的版本控制系统，git出现之前，Linux的源码都是托管在一个叫做BitKeeper的版本控制系统之上的，不过由于Linux开发者试图破解BitKeeper的协议导致BitKeeper公司的不满，于是收回了Linux免费使用BitKeeper的权限。Linus大怒，于是自己花了两周开发了一套版本控制系统，就是最早期的Git，牛就一个字！

## Git与传统版本控制系统（如SVN）的区别

传统版本控制系统一般就是指集中式控制系统，版本库在中央服务器中存放，我们需要先从中央服务器中拿到代码，再进行编辑后重新上传到中央服务器中。这里系统最大的问题是需要联网才能工作，如果网络有故障或者网速很慢，会比较蛋疼。

以Git为代表的分布式版本控制系统去掉了中央服务器的概念，每个人的电脑就是自己的"中央服务器"，你只需要把需要进行版本控制的代码放到本地电脑，不需要联网就能工作了。那你可能会问，只在自己电脑上进行版本控制，那怎么进行多人协作呢？别急，要进行多人协作只需要连根网线，把各自的修改推送给对方就可以了。

集中式和分布式版本控制系统的区别主要在于，分布式版本控制系统安全性要更高一些，因为每个人的电脑中都有一份完整的代码库，任何一个人的电脑坏掉都不要紧，从其他人电脑中拉取一份就可以了。但是集中式版本控制系统如果中央服务器挂了，所有人就都别干活了。

Git在实际使用中，很少在两个人之间直接推送版本库的修改，而是使用一台公共主机作为中间存储机，每个人都将自己的修改推送到这台中间机上，同时从这台中间机上更新代码到本地上来。

类似于这样：

![1][1]

另外Git还有强大的分支管理系统，远比SVN优秀。

Git还免费！

## Git安装

只讲以Ubuntu为代表的Linux系统下的安装方法吧，其他系统自行google吧。

一个命令搞定：

```
sudo apt install git
```

## 基本操作

1. 配置git

    一些标识信息：

    ```
    git conf --global user.name "jony"
    git conf --global user.email "jony@jony.com"
    ```

    这里的global表示的是配置级别，Git一共有三个配置级别：

    - --local 【默认,高优先级】只影响本地仓库。配置内容会放在当前文件夹的config文件中：`.git/config`

    - --global 【中优先级】会影响到所有当前用户的Git仓库，配置内容存放在`~/.gitconfig`

    - --system 【低优先级】影响系统所有的git仓库，配置内容存放在 `/etc/gitconfig`

2. 初始化仓库init

    ```
    ➜  git_test git:(master) tree -a
    .
    └── .git
        ├── HEAD
        ├── branches
        ├── config
        ├── description
        ├── hooks
        │   ├── applypatch-msg.sample
        │   ├── commit-msg.sample
        │   ├── fsmonitor-watchman.sample
        │   ├── post-update.sample
        │   ├── pre-applypatch.sample
        │   ├── pre-commit.sample
        │   ├── pre-push.sample
        │   ├── pre-rebase.sample
        │   ├── pre-receive.sample
        │   ├── prepare-commit-msg.sample
        │   └── update.sample
        ├── info
        │   └── exclude
        ├── objects
        │   ├── info
        │   └── pack
        └── refs
            ├── heads
            └── tags
    10 directories, 15 files
    ```

3. 查看版本库状态git status

    在介绍git status命令之前需要明白git进行版本控制的基础，git将文件状态分为未跟踪和已跟踪两部分，另外将版本库在不同状态下又分为三个工作区域，分别是工作区、暂存区和提交区。
    
    Git是先将工作区也就是版本库所在目录下的文件加入到暂存区，当暂存区的文件添加到一定量时由用户再将暂存区的文件一并加入到提交区，进入提交区的文件就可以被Git管控，进行版本控制了。
    
    另外加入远程地址后，还可以把提交区的文件内容推送到远程仓库，从而与他人协作开发。

    ![2][2]

    git status就是用于查看当前版本控制库的状态的命令，通过这个命令可以看到你当前的版本库到底处于上图中的那个状态下。

    我切换到了另一个自己的git项目下，使用`git status`命令查看当前版本库状态：

    ```
    ➜  pics git:(master) git status 
    On branch master
    Your branch is up-to-date with 'origin/master'.
    nothing to commit, working directory clean
    ```

    在该目录下添加一个文件：

    ```
    ➜  pics git:(master) touch a
    ➜  pics git:(master) ✗ git status 
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Untracked files:
    (use "git add <file>..." to include in what will be committed)

            a

    nothing added to commit but untracked files present (use "git add" to track)
    ```

4. 使用`git add`添加工作目录下的文件到暂存区

    如上所说，git想要将文件纳入版本控制中，就需要将文件添加到版本库中，添加的命令就是`git add`。

5. gitignore文件

    如果在当前目录下不想添加某些或某种类型的文件，就可以通过编写一个.gitignore文件，将不想添加的文件名或某种类型文件的通配名加入到.gitignore文件中即可，需要记住的是.gitignore文件需要加入到版本控制中去。

    这就相当于在进行`git add`操作时，进行了一层gitignore的过滤。

6. 从暂存区删除文件`git rm`

    - `git rm --cached`：仅从暂存区删除。

    - `git rm`:从暂存区和工作去都删除相关文件。

    - `git rm $(git ls-files --deleted)`:小技巧，删除所有被跟踪，但是在工作目录被删除的文件。

7. 提交暂存区的文件`git commit`

    `git commit`用于将暂存区的内容提交到版本控制系统上，提交之后的内容才能被版本控制记录。

8. 查看历史提交记录`git log`

    使用`git log`可以查看详细的历史提交记录，使用`git log --oneline`可以简要查看提交历史记录。

9. 自定义git命令(别名)

    比如自定义`git log`的查看提交记录的命令`git lg`
    ```
    git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
    ```

    使用`git log`查看提交记录：

    ```
    * c9e7b28 - (HEAD -> master, origin/master, origin/HEAD) add git_dis (2 hours ago) <jiaoqiyuan>
    * 11c787c - add git_dis (2 hours ago) <jiaoqiyuan>
    * 15aa062 - add git_dis (2 hours ago) <jiaoqiyuan>
    * 5037e89 - add git_note dir (13 hours ago) <jiaoqiyuan>
    * 78fa4d8 - Add files via upload (13 hours ago) <Qiyuan Jiao>
    * 05fa4f6 - Create README.md (13 hours ago) <Qiyuan Jiao>
    ```

    根据--global参数我们可以知道，这个配置是存放在`~/.gitconfig`中的:

    ```
    [user]
        email = jiaoqiyuan@outlook.com
        name = jiaoqiyuan
    [alias]
            lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    ```

10. 显示版本差异`git diff`

    - `git diff`：显示工作区与暂存区的差异。

    - `git diff --cached [<reference>]`：显示暂存区与某次提交的差异，默认是HEAD。reference表示某次提交版本的SHA-1编码。

    - `git diff <reference>`：显示工作区与某次提交的差异。

11. 撤销本地修改`git checkout`

    `git checkout -- <file>`: 撤销文件的修改。

12. 撤销暂存区的修改`git reset`

    `git reset HEAD <file>`：将文件在暂存区的修改撤销掉，恢复到工作区。

13. 撤销全部改动`git checkout HEAD -- <file>`

    直接撤销暂存区的文件并恢复到工作目录之前的状态。

14. 直接将工作区的文件提交到提交区`git commit -a`

    将文件更改后直接提交，省去添加到暂存区的过程。

![3][3]

## 分支操作

分支操作是git的另一大优秀特性，也是我面试进坑的地方，平时没怎么使用git写作开发，分支操作不是很熟悉，这里记录一下。

1. 分支的增删改查`git branch`

    - 创建一个分支：`git branch <branchName>`

    - 删除一个分支：`git branch -d <branchName>`

    - 显示所有分支信息：`git branch -v`

    - 切换分支：`git checkout <branch>`

2. reset和checkout的区别
 
    - commit操作：reset是回退到某个版本，HEAD的branch（分支版本）都移动到指定分支；checkout只是切换分支，只移动HEAD，branch(分支版本)不变。

    - file操作：reset作用于文件时是将文件从暂存区恢复到工作区，一般是add后使用reset将文件恢复到add前的状态；checkout用于将工作区的文件回到某状态，一般是修改文件内容后将文件内容恢复到修改前的状态。

3. 保存当前分支状态`git stash`，返回到干净的工作空间

    用于保存当前分支状态，使用户在没有提交当前修改的情况下也能切换到其他分支，这在开发过程中是很有用的，比如在当前分支正在编码过程中有其他紧急任务需要处理，这时又不能直接提交当前的共作，但是又需要切换到其他分支，这时就可以使用`git stash`。

    先使用`git stash save "save my work now"`

    再切换到其他分支，工作完成后再将之前保存到stash的内容检出`git stash apply stash@{0}`，然后删除stash`git stash drop stash@{0}`。

    这两步可以合为一步，直接使用`git stash pop stash@{0}`即可。

    ![4][4]

4. 合并分支`git merge`

    特性分支完成开发后需要合并到主分支，这时使用`git merge`命令完成。

    一般在有远程服务器的场景下常用的使用是：`git fetch origin master`，然后`git merger origin master`。也可以直接使用git pull一步完成：`git pull origin master`。

  
## 总结

git的学习重在使用，github官方有几个不错的练手地方，[点击这里][5] 查看。

阮一峰老师总结了一些git常用方法，[点击这里][6]查看，总结得很好，常用命令一般就这几个：

![7][7]

还有廖雪峰老师的git教程，很不错，[点击这里][8]查看。

另外还有一些git的使用方法没介绍，例如`git rebase`，自己多查查吧。



[1]: https://github.com/jiaoqiyuan/pics/raw/master/git_note/git_dis.jpeg
[2]: https://github.com/jiaoqiyuan/pics/raw/master/git_note/git_status.png
[3]: https://github.com/jiaoqiyuan/pics/raw/master/git_note/git_workspace.png
[4]: https://github.com/jiaoqiyuan/pics/raw/master/git_note/git_stash.png
[5]: http://try.github.io/
[6]: http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html
[7]: http://www.ruanyifeng.com/blogimg/asset/2015/bg2015120901.png
[8]: https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/