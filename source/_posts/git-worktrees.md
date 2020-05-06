---
title: Git WorkTrees：你从未听说过的 Git 最佳特性 - 译
date: 2020-05-06 15:08:29
tags: git
categories: 翻译
thumbnail: https://git-scm.com/images/branching-illustration@2x.png
photos: https://git-scm.com/images/branching-illustration@2x.png
---

信息来源：[阮一峰每周分享](https://github.com/ruanyf/weekly/blob/master/docs/issue-105.md)

文章来源：[Discussing Docker. Pros and Cons.](https://levelup.gitconnected.com/git-worktrees-the-best-git-feature-youve-never-heard-of-9cd21df67baf)

----

### Before There Were Trees 在有 worktree 之前

When I first started using git professionally, one habit I had was to frequently stash code. I’d be working on a particular feature and I’d see a Hipchat alert (yes, this was before Slack) about an error being thrown in production. I’d take a look at the stacktrace in Sentry, and find the offending section of code. From there I’d return to my terminal, stash my changes, open a new bugfix branch off master, and begin trying to reproduce the error locally.

我最初使用 git 时，有一个习惯就是经常使用 `stash` (暂存) 代码。我正在开发一个特定功能，并且会看到有关在生产中引发错误的 Hipchat 警报（对，这是在 Slack 之前）。***这里的 Hipchat 和 Slack 是两个聊天工具，就想象成我们熟悉的微信或者 QQ 也行。*** 我需要看一眼哨兵中的堆栈信息，找到有问题的那部分代码。在这之后我需要回到终端，暂存我的修改，在 master 分支上打开一个新的 bugfix 分支，然后尝试在本地尝试复现这个错误。

Usually this process would happen without too much pain. If there were untracked files in my working tree, I’d just add them and stash everything. After I’d pushed a bugfix branch up to Github, I’d unstash my changes and unstage any of the original untracked files I had added. Occasionally I’d make a mistake and do something awful like accidentally trashing my untracked files, or uncommitted changes.

通常情况下这个操作不会让人感觉太痛苦。如果 git 工作空间中有未跟踪的文件，我只需要 add 一下这些文件然后 stash 所有修改就可以了。在我推送了一个 bugfix 分支到 Github 后，使用 unstash 取消我之前暂存的修改，而且 unstage 我之前使用 add 添加的未跟踪的文件就可以。有时我会犯个错误，并且做一些糟糕的事情，例如不小心丢掉了我的未跟踪文件或未提交的更改。

Other times, I’d be working on a feature, and I’d see a PR review request from one of my coworkers. I generally try to review code as fast as possible, and depending on how urgent the change was, I would stop what I was working on to review the changes. Again, this meant stashing my changes and pulling down the code locally to try out the new changes, and verify edge case behavior.

其他时候，当我正在开发一项功能时，可能会看到一位同事的 PR 审查请求。通常我会尽可能早地审查代码，而且根据 PR 修改的紧急程度，我可能需要停止手头的事情来检查修改的代码。同样地，这意味着我需要暂存我的修改然后拉取远端代码到本地，运行涉及修改的那部分代码的功能，并验证一些边界情况。

Eventually, I got frustrated with the pain of stashing and unstashing, and decided that I should just clone the repository into another directory, and I would use that copy of the repository for bugfixes and code reviews. And this mostly worked. However, it isn’t without its downsides.

最终，在 stash 和 unstash 的痛苦过程中我感到沮丧，我觉得我应该在其他目录再 clone 一个项目，使用这个目录下的项目进行 bug 修复和代码审查。而且这通常是有效果的。 但是，这也有一些弊端。

One big issue with having multiple copies of a repo in different directories is that branches aren’t shared between them. Sure, you can set one directory to be a remote of the other, but it’s still painful to constantly be syncing branches. In practice, if you use a development model where all code gets pushed to a central repository like Github, this isn’t as painful. Similarly, if you use stashing a lot, as I did, it takes a tiny bit of work to apply a stash from my directory to the other as a patch.

一个很大的问题是在不同不目录下使用工程的多个拷贝的不同分支之间是无法共享数据的。当然，你可以将一个目录设置为另一个目录的远程目录，但是持续同步分支仍然很麻烦。实际情况下，如果你使用一个像 Github 的开发模式，将所有代码都推送到到一个中心仓库，这还不算多让人痛苦。与之相似，如果你经常像我那样使用 stash 暂存修改，将我目录中暂存的修改作为补丁应用到另一个目录则需要亿点点工作量。

Another issue is that each directory has its own .git directory. This means all sorts of configuration has to be duplicated. I use a fair number of git pre-commit hooks, so these now needed to exist in both directories. Again, there are ways to work around this, such as using symlinks, or possibly centralizing some of this config in your home folder’s .gitconfig, but it’s not ideal.

另一个问题是每个目录都有自己的 `.git` 目录，这表示所有配置都需要在各个目录下重复进行。我使用了大量的 git pre-commit 钩子，因此现在两个目录中都必须存在这些钩子。同样，有一些方法可以解决这个问题，例如使用符号链接，或者将某些配置集中在主文件夹的 `.gitconfig` 文件中，但是这样做并不理想。

Finally, this process didn’t easily scale beyond the 2 concurrent repo case. When I found myself needing to halt my existing work, create a bugfix, and review a coworkers code all at the same time, I was faced with the prospect of cloning another copy of the repo. Cloning the repo wasn’t exactly quick, and again I was faced with the burden of copying and keeping in sync all the configuration I had for the existing two repositories.

最后一个问题，在同时处理两个以上的项目时会比较困难。当我同时需要停止当前的工作，创建一个 bugfix 分支，review 同事的代码时，我就想还是再 clone 一个项目到本地吧。clone 一个项目也是需要花费时间的，然后我又要面对拷贝带来的问题以及配置另外两个项目中使用到的 git 配置。


### Enter Git Worktrees 使用 Git Worktrees

Git Worktrees are a feature that allow you to have a single repository with multiple checked out working branches at the same time. Now that may not sound that cool, but let me lay out an example.

Git Worktrees 这个特性让你在只拥有一个项目仓库的情况下可以 checkout 出多个工作分支。现在听起来可能并不是那么酷，那么让我举一个例子吧。

I have a copy of the `ApertureScience` repository located at `/home/James/Aperture`. I’m going to begin working on a new feature, `cake`. Before I start this work, I’m first going to run `git worktree add cake`. This will create a new directory for me to do my work located at `/home/James/Aperture/cake`. I spend a few hours working on my `cake` feature.

在 `/home/James/Aperture` 目录下我有一个 `ApertureScience` 仓库的副本，我打算在一个新的分支 `cake` 上展开工作，在开始工作前，我需要首先执行 `git worktree add cake`，为我接下来工作创建一个新的目录 `/home/James/Aperture/cake`，然后我就可以在接下来的几个小时里为 `cake` 特性编写代码了。

A wild error appears in our production app. Probably a rogue test subject causing trouble… Better go deal with that.

我们的生产应用中出现了一个严重错误。可能是一个测试对象导致的问题……最好去解决这个问题。

I stop working on the `cake` feature and run the command `git worktree add bugfix` inside the `/home/James/Aperture` directory. This almost instantly creates a new working copy of the repository at `/home/James/Aperture/bugfix` on a new branch called `bugfix`. I cd into that directory and run whatever commands I need to boot up the application. Once I discover the bug and figure out what changes need to be made to fix, I commit the changes, push up my branch for review, and then return to the feature work I was doing on `/home/James/Aperture/cake`. No stashing hassle, and no worrying that my most recent pre-commit hooks aren’t running.

这时我停止开发 `cake` 特性，然后在`/home/James/Aperture` 目录下执行 `git worktree add bugfix`，执行这个命令后几乎可以立即在 `/home/James/Aperture/bugfix` 目录下创建一个新的名叫 `bugfix` 的拷贝了当前工作环境的分支，我 `cd` 进这个目录后可以执行任何我启动应用需要的命令。一旦发现错误并找出需要进行哪些更改才能修复，我提交更改，向上推我的分支进行审核，然后返回 `/home/James/Aperture/cake` 目录继续进行 `cake` 特性的开发。没有 stash 带来的麻烦，也不需要担心我的 pre-commit hooks 不工作的问题。

Effectively, you can work on as many things in parallel as you’d like and can feel free to just pause your work and come back later.

实际上，您可以根据需要并行处理许多事情，并且可以随时暂停工作，稍后再回来继续之前的工作。

Why aren’t more people talking about Git Worktrees?? They’re incredible!

为什么没有更多的人谈论 Git Worktrees ？？ 他们太不可思议了！

### Can You Really Have Your Cake and Bugfix Branches Too? 你真的有 cake 和 bugfix 分支吗？

Of course, there are some downsides to be aware of when using git worktrees.

当然，使用 git worktrees 时要注意一些缺点。

If you’re not stashing your code, or being forced to commit it, you’re more likely to just leave code in a working state. Usually, this is fine, but it means that you may not have any refs in the reflog if something goes wrong, or you may be less likely to have a committed copy of your code somewhere if you suffer a disk failure.

如果你没有暂存你的代码，或者被强制要求 commit，你可能只是将代码置于工作状态，通常情况下这是没问题的，但这也意味着在出现问题时你无法通过 reglog 来记录一个操作，或者如果你遇到磁盘故障，则不太可能在某处拥有已提交的代码副本。

It’s possible that being able to work on more things at once may also be detrimental to your productivity. You might wind up splitting your focus and try to do too many things in parallel.

一次可以处理更多事情可能也会降低你的工作效率。你可能会分散注意力，并尝试并行执行太多操作。

Finally, some tools may not fully support worktrees well. Worktrees rely on the fact that git allows .git to be a file that points to a directory, instead of an actual directory. Some tools incorrectly assume that .git must be directory and run into issues with worktrees.

最后，一些工具还没有很好地支持 worktrees，Worktrees 依赖于 git 允许 `.git` 作为指向目录的文件，而不是一个真实的目录。一些工具错误地认为 `.git` 必须是目录，并且使用 worktrees 会遇到一些问题。

### Recommended Setup 推荐设置

I tend to try to keep things tidy, so I have my worktrees setup in a specific fashion to allow me to work easily. My worktrees folder looks like the following:

我趋向于保持一切整洁，所以我以特定的方式设置了工作树来帮助我轻松工作。我的 worktrees 目录大概像下面这个样子：

`/home/James/worktrees/.bare` — This is the actual git repository, a bare instance. This stores only the repo metadata and has no actual working tree. I keep this in a separate hidden directory so that I can create new worktrees from this folder, but don’t accidentally try to check out a branch in the `worktrees` directory, which is an operation that can only be done in a working tree.

`/home/James/worktrees/.bare` --这是实际的git仓库，一个裸实例。这里仅存储仓库元数据，而没有实际的工作树。我将其保存在一个单独的隐藏目录中，以便可以在这个文件夹中创建新的工作树，但不要意外地尝试检出 worktrees 目录中的分支，该操作只能在工作树中完成。

`/home/James/worktree/.git` — This file just points to the `.bare` directory.

`/home/James/worktree/.git` — 该文件仅指向 `.bare` 目录。

`/home/James/worktree/master` — I always keep a copy of master, mostly so that if something comes up I can spin up a local copy of whatever is running in production.

`/home/James/worktree/master` — 我通常会保留一份 master 副本，主要是为了在出现问题时可以在生产环境中运行任何本地副本。

`/home/James/worktree/hotfix` — This branch is kept as a place where I perform hotfixes. It happens with enough frequency that I keep a dedicated worktree around for it.

`/home/James/worktree/hotfix` — 该分支保留为我执行 bugfix 的地方。bugfix 频繁发生，我为此保留了一个专用的工作树。

`/home/James/worktree/<feature>` — I have a number of different feature worktrees, which are divided by particular feature areas of the codebase.

`/home/James/worktree/<feature>` — 我有许多不同的功能工作树，它们按代码库的特定功能区域划分。

I’ve tried a couple different schemes for how to organize these branches, and still think there’s room for improvement, but am fairly happy with having at least a `master` worktree that allows easily spinning up exactly what is live on master, and a number of feature branches that allow me to work on multiple things in parallel. I also like to keep around worktrees such as, `dependencies` where I can drop in, bump a dependency, and spend a few minutes working on getting our code base to work with a new version of the dependency.

我已经尝试了几种不同的方案来组织这些分支，但仍然认为还有改进的余地，不过我对至少拥有一个主工作树很满意，该工作树可以轻松地将主工作完全分解，以及许多功能分支，这些分支允许我并行处理多个事情。我还喜欢保留工作树，例如可以插入的依赖项，增加依赖项，并花几分钟时间来使我们的代码库与新版本的依赖项一起使用。

As a bonus, if you use Docker-Compose, or any other tool that is isolated by directory path, you can have multiple copies of your application running at the same time. Docker-Compose relies on using it’s containing folder for as a name for the containers that are created and for managing running containers. Of course, to make this work, you need to make sure your application is able to properly isolate itself with dynamic port allocation, etc.

另外，如果使用 Docker-Compose 或按目录路径隔离的任何其他工具，则可以同时运行应用程序的多个副本。Docker-Compose 依靠将其包含文件夹用作创建的容器的名称以及管理运行中的容器。当然，要使其正常工作，你需要确保你的应用程序能够通过动态端口分配等适当地隔离自身。

All in all, this style of working has made it so much easier for me to context switch when it is necessary, and provides me with small chunks of work that I can get done in less than 1 hour increments.

总而言之，这种工作方式使我在需要时可以更轻松地进行上下文切换，并且为我提供了可以不到 1 小时就完成的小块工作。

I consider git worktrees to be one of the secret weapons in my toolbox and could not imagine going back to developing without them.

我认为 git worktrees 是我工具箱中的秘密武器之一，无法想象没有它们该如何回过头去进行那些开发工作。