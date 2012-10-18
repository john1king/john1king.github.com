---
layout: post
title: Git command 
---


添加文件
--------

添加或更新所有文件

{% highlight html %}
  git add -A
{% endhighlight %}

添加修改和删除的文件到缓存，不会添加新新建的文件

{% highlight html %}
  git add -u
{% endhighlight %}

git ls-files
------------

列出没有添加到版本库的文件

{% highlight html %}
  git ls-files --other --exclude-standard
{% endhighlight %}

可以添加下面的 alias 到 `~/.gitconfig` 中

{% highlight html %}
  au = !git add $(git ls-files -o --exclude-standard)
{% endhighlight %}

添加包含匹配模式的文件

{% highlight html %}
  git ls-files -m | grep <pattern> | xargs git add
{% endhighlight %}

**参考链接**

- [Git: list only “untracked” files (also, custom commands)](http://stackoverflow.com/questions/3801321/git-list-only-untracked-files-also-custom-commands)


分支
----

以非快进的方式合并分支并编辑 commit 信息

{% highlight html %}
  git merge <branch1> <branch2> --no-ff --edit
{% endhighlight %}

日志
----

显示被修改文件的统计信息， `--stat` 显示每个文件的修改信息， `--shortstat` 只显示 `commit` 的修改信息

{% highlight html %}
  git log --stat 
  git log --stat --oneline
  git log --shortstat --oneline
{% endhighlight %}

查看每个提交者的 commit 数量， `-s` 不显示 commit, `-n` 指定统计的 commit 总数

{% highlight html %}
  git shortlog [-sn]
{% endhighlight %}

撤销操作
--------

迁出文件

{% highlight html %}
  git checkout .
  git checkout <file1> <file2>
{% endhighlight %}

撤销添加到缓存的文件

{% highlight html %}
  git reset 
  git reset . 
{% endhighlight %}

放弃合并分支。`reflog` 查看操作历史，`reset` 回到制定操作的状态

{% highlight html %}
  git refog
  git reset --hard HEAD@{2}
{% endhighlight %}

反向提交，创建一个撤消上次提交(HEAD)的新提交

{% highlight html %}
  git revert HEAD
{% endhighlight %}

打包
----

打包当前分支代码到 zip 文件

{% highlight html %}
  git archive --format zip -o filename.zip HEAD
{% endhighlight %}

用日期作为打包后的文件名

{% highlight html %}
  git archive --format zip -o $(git log --date=short --pretty=format:"%ad" -1).zip HEAD
{% endhighlight %}

**参考链接**

- [git archive 與 log 小技巧](http://people.debian.org.tw/~chihchun/2010/02/01/git-archive-and-log/)

其它
----

在 gui 中查看所有分支的历史

{% highlight html %}
  gitk --all 
{% endhighlight %}
