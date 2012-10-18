---
layout: post
title: Git command 
---


添加文件
--------

添加或更新所有文件
```
  git add -A
```

添加修改和删除的文件到缓存，不会添加新新建的文件
```
  git add -u
```

git ls-files
------------

列出没有添加到版本库的文件
```
  git ls-files --other --exclude-standard
```

可以添加下面的 alias 到 `~/.gitconfig` 中
```
  au = !git add $(git ls-files -o --exclude-standard)
```

添加包含匹配模式的文件
```
  git ls-files -m | grep <pattern> | xargs git add
```

### 参考链接:
- [Git: list only “untracked” files (also, custom commands)](http://stackoverflow.com/questions/3801321/git-list-only-untracked-files-also-custom-commands)


分支
----

以非快进的方式合并分支并编辑 commit 信息
```
  git merge <branch-1> <branch-2> --no-ff --edit
```

日志
----

显示被修改文件的统计信息， `--stat` 显示每个文件的修改信息， `--shortstat` 只显示 `commit` 的修改信息

```
  git log --stat 
  git log --stat --oneline
  git log --shortstat --oneline
```

查看每个提交者的 commit 数量， `-s` 不显示 commit, `-n` 指定统计的 commit 数量
```
  git shortlog [-sn]
```

撤销操作
--------

迁出文件
```
  git checkout .
  git checkout <file1> <file2>
```

撤销添加到缓存的文件
```
  git reset 
  git reset . 
```

放弃合并分支。`reflog` 查看操作历史，`reset` 回到制定操作的状态
```
  git refog
  git reset --hard HEAD@{2}
```

反向提交，创建一个撤消上次提交(HEAD)的新提交
```
  git revert HEAD
```

打包
----

打包当前分支代码到 zip 文件
```
  git archive --format zip -o filename.zip HEAD
```

用日期作为打包后的文件名
```
  git archive --format zip -o $(git log --date=short --pretty=format:"%ad" -1).zip HEAD
```

### 参考链接
- [git archive 與 log 小技巧](http://people.debian.org.tw/~chihchun/2010/02/01/git-archive-and-log/)

其它
----

在 gui 中查看所有分支的历史
```
  gitk --all 
```
