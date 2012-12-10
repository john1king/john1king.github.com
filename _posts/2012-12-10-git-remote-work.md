---
layout: post
title: Git 推送远程分支
---


使用 git 同步工作代码，经常会遇到远程仓库版本和本地分支差异非常大的情况。例如对一些特性分支执行了多次 `rebase` 或 `commit --amend` ，修改了分支名称等。遇到这种情况，简单的 `git push origin branch` 已经无法满足需求了。

解决这个问题最直接了当的方法是镜像远程仓库

{% highlight html %}
  # 客户端1
  git push origin --mirror

  # 客户端2
  git pull origin -f
{% endhighlight %}

这种方式威力过于巨大，需要慎重使用。实际上只要掌握 git push 的几个用法，足以解决大部分需求。在 git 中真正决定代码历史的是 commit，而分支只是标识 commit 的手段，因此能够对分支进行非常丰富的操作。

{% highlight html %}
  git push origin :
  git push origin +:
{% endhighlight %}

这两条命令推送所有存在于远程仓库的同名本地分支，`+` 相当于 `--force`，强制推送非非快进合并的分支。注意这里推送的分支时远程仓库**最新**的分支状态，而不是上次 pull 时的状态。 默认配置下 `git push origin` 相当于 `git push origin :`，因此在第一次推送到新建分支时需要指定分支名称，而之后可以则省略。不过推送时指定分支名还是是比较好的实践，可以避免推送不想更新的分支。

如果想要删除远程仓库的分支，需要指定分支名

{% highlight html %}
  # 删除远程分支 br1 和 br2
  git push origin :<br1> :<br2>
{% endhighlight %}

重名远程分支则需要组合删除和推送的操作

{% highlight html %}
  git branch -m old_branch_name new_branch_name
  git push origin new_branch_name
  git push origin :old_branch_name
{% endhighlight %}


一些实践
--------

1. 保留 master 分支。git 的很多操作默认基于 master， 如 git clone 等， 保留 master 分支会省去很多麻烦。

2. 初始化 commit。没有 commit 很多操作都无法执行，如 `reset`, `gitk` 等。在新建项目时做一个“空”的提交，内容可以是 `.gitkeep` 或者 `README` 等

3. 慎重合并主要分支。这里的主要分支指定是类似但部分项目的 dev 和 master 等分支。即便是是个人项目，不修改已经提交到主要分支上的 commit，保持这些分支的稳定也是一个很好的实践。不执着于完美的分支线索，不断的向前提交，使用 git 才能够带来最大的效率提升。

4.工作在特性分支上。如果忘记新建分支，在主要分支上提交了 commit, 可以通过 `git branch -f` 移动分支到原来的位置。在特性分支上没有主要分支上的顾虑，大胆的 reabse ，只要在合并到主要分支前检查一下即可。

{% highlight html %}
  git checkout -b feature
  git branch -f dev origin/dev
{% endhighlight %}

