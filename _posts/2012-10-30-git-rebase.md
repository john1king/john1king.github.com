---
layout: post
title: 使用 git rebase 交互模式整理提交历史
---

在 git 中，基于分支和频繁提交是最常见的工作模式，这样很容易留下一些混乱或无用的 commit，让查看代码记录时变得十分不便。构建干净的提交历史可以使用 rebase 命令线性的合并分支和整理已提交的 commit。 
rebase 的详细用法在帮助文档和文末给出的参考链接中都有详细的介绍，本文只探讨几种 rebase 交互模式常见的应用场景。

{% highlight html %}
  git rebase -i <commit>
{% endhighlight %}

使用 `-i` 选项进入交互模式，git 在执行 rebase 操作前会弹出编辑器窗口，列出将会处理的 commit 列表。由于交互模式更常用于整理 commit，所以命令中的 `<commit>` 通常是当前分支上的一个 commit

{% highlight html %}
  pick d51794a b
  pick 4554fa7 c
  pick 4bfcb0f d
  pick a875c8a e

  # Commands:
  #  p, pick = use commit
  #  r, reword = use commit, but edit the commit message
  #  e, edit = use commit, but stop for amending
  #  s, squash = use commit, but meld into previous commit
  #  f, fixup = like "squash", but discard this commit's log message
  #  x, exec = run command (the rest of the line) using shell
  #
  # These lines can be re-ordered; they are executed from top to bottom.
  #
  # If you remove a line here THAT COMMIT WILL BE LOST.
  # However, if you remove everything, the rebase will be aborted.
  #
{% endhighlight %}

可以看到注释中已经把基本用法都列举出来了，在交互模式中可以方便的做到 commit 的删除，排列和修改。例如

{% highlight html %}
  e d51794a b
  s 4554fa7 c
  s 4bfcb0f d
  s a875c8a e
{% endhighlight %}

合并了 `b`, `c`, `d` 三个提交，并修改 `b` 的提交信息。这里没有使用 `fixup` 选项，`fixup` 将当前 commit 合并上一 commit 中，但保留的是上一个 commit 的提交信息，并不是很实用。

### 拆分提交

拆分提交的操作要稍微麻烦些。还是上面的例子，假设在 `d` 提交中添加了两个文件 `d1` 和`d2`，现在需要将添加两个文件的修改分开到两个提交中。首先使用 `edit` 选项

{% highlight html %}
  pick d51794a b
  pick 4554fa7 c
  e 4bfcb0f d
  pick a875c8a e
{% endhighlight %}

git 在执行到 `d` 时，会停下来等待用户操作。这时候撤消 `d` 的提交，并分别提交 `d1` 和 `d2`

{% highlight html %}
  git reset head^
  git add d1
  git commit -m d1
  git add d2
  git commit -m d2
{% endhighlight %}

最后执行 `continue` 命令完成 rebase

{% highlight html %}
  git rebase --continue
{% endhighlight %}

### 交换位置

在交互模式中只要修改 commit 的顺序就可以交换位置。不过需要注意一些 commit 间存在依赖关系，处理不当会需要处理很多额外的冲突。例如在 `a` 中添加了一行代码，在 `b` 中又修改了这行代码，如果将 `b` 移动到 `a` 前就会导致在 rebase 过程中出现冲突。

除非必须，尽量的少移动 commit 的位置。不过两种情况 commit 可以很容易的移动 commit：几个 commit 修改的都是不同的文件；commit 的改动很小，只是修改了部分拼写错误。

### 插入新提交

可以使用 `edit` 选项来插入新的提交。不过当新的修改和之前的不冲突时，更简单的方式是先添加，再移动到想要的位置。继续上面的例子，在拆分 完 `d` 之后，发现还想添加一个文件 `d3`

{% highlight html %}
  touch d3
  git add d3
  git commit -m d3
{% endhighlight %}

修改 commit 顺序后保存即可

{% highlight html %}
  pick d51794a b
  pick 4554fa7 c
  pick 4bfcb0f d1
  pick e17b61b d2
  pick 827b329 d3
  pick a875c8a e
{% endhighlight %}

### 删除远程分支

整理完分支之后，之前提交到远程仓库的分支已经没有用处了。可以执行以下命令删除远程仓库中对应的分支

{% highlight html %}
  git push origin :branch_name
{% endhighlight %}

语法比较奇怪，意思是提交一个空分支到远程仓库，从而起到删除远程分支的作用。

### 恢复到之前的状态

标准的方法是使用 `reflog` 打印操作历史， `reset` 命令恢复到指定的步骤

{% highlight html %}
  git reflog
  git reset HEAD@{10}
{% endhighlight %}

rebase 的分支 commit 比较多的时候要找到之前的操作在哪一部并不容易。更容易理解的方式为 rebase 前备份一个分支，放弃 rebase 时只要删除当前分支即可

{% highlight html %}
  git checkout -b feature_b
  git rebase master feature
  git checkout feature_b
  git branch -D feature
  git branch -m feature_b feature
{% endhighlight %}

### 总结

- 经常执行 rebase 保持分支历史整洁，每次 rebase 的 commit 数量尽量少些，这样更容易控制
- 不要 rebase 已经发布的分支

参考链接
--------
- [Git rebase 和 merge 合併操作示範錄影](http://ihower.tw/blog/archives/6704/)
- [Git-rebase 小筆記](http://blog.yorkxin.org/2011/07/29/git-rebase)
