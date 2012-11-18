---
layout: post
title: Windows 下使用 gVim 编辑远程文件
---

### 为什么要使用远程编辑

1. Windows 下缺乏一个好用的 SSH 客户端工具
2. 使用 SSH 时不便于复制粘贴
3. 已经对本地的 vim 进行大量的配置，安装了众多插件

### 基本用法

在使用 vim 编辑远程文件时，scp 是最常用的传输方式。使用方式如下

{% highlight html %}
  :edit scp://username@hostname//root/.ssh/authorized_keys 
{% endhighlight %}

注意命名中的双 `/`，如果只有单个 `/`，表示编辑用户目录下的问题。两个 `//` 表示文件路径从更目录开始

### 安装与配置

遗憾的是 Windows 下并没有原生的 scp 工具可用，需要自己安装。比较常见的选择是 Cygwin 和 PuTTY，不过本文使用的是 msysgit 安装后自带的 scp 工具，如果已经安装了这个最常用的 windows 平台的 git 版本，会比其它两种方式方便很多。Cygwin 是个大家伙，PuTTY虽然能够比较好的支持 Windows 路径，但使用的 ssh key 有些古老，需要额外的转换操作。

在开始前，请确保 msysgit 的 bin 目录已经添加到系统的环境变量 `Path` 中。该目录通常在 msysgit 安装目录下，包含了很多 unix 工具的不完全移植版本。可以在 git bash 中使用 scp 命令

{% highlight html %}
  scp root@192.168.1.11:.ssh/authorized_keys /C/tmp/authorized_keys
{% endhighlight %}

msysgit 的 scp 命令不能用识别 Window 下的 `C:` 格式的盘符，需要将其转换为 `/C/` 格式。vim 的 netrw 插件并不直接支持这种格式的路径，默认情况下 netrw 把临时文件保存在操作系统的临时文件下（cmd 中执行 `set TEMP` 可获取该目录）。因此编辑远程文件时会出问题。所幸的是 netrw 提供了对 cygwin 的支持，对其进行稍加修改就能够支持 msysgit 的 scp。

1. 打开 `$VIM\vim73\autoload\netrw.vim`
2. 替换所有的 `cygdrive/` 为 `/` 后保存，也许你应该在替换前确认到底做了哪些修改

{% highlight html %}
 :%s/cygdrive\///g` 
 :wq
{% endhighlight %}

3. 配置 vimrc , 使 cygwin 模式。如果系统临时文件目录比较长，名称中包含 `~` 等字符，还需要修改临时文件的目录 `$TMP`。vim 生成的临时文件名可以使用 `:echo tempname()` 查看

{% highlight html %}
  let g:netrw_cygwin = 1
  let $TMP='C:\tmp\'
{% endhighlight %}

到此已经可以正常使用 scp 编辑远程文件了。如果已经配置好 ssh key，可以加入配置

{% highlight html %}
  let g:netrw_silent = 1
{% endhighlight %}

这样每次下载和上传文件后后将自动关闭shell窗口

参考链接
--------
- [Using Putty on Windows to login Linux securely via OpenSSH](http://linux-sxs.org/networking/openssh.putty.html)
- [Copying remote file to a specified path on local desktop using scp](http://superuser.com/questions/291840/copying-remote-file-to-a-specified-path-on-local-desktop-using-scp)
- [Transparently edit remote files on Windows, with ssh/Putty and netrw](http://stackoverflow.com/questions/3546819/transparently-edit-remote-files-on-windows-with-ssh-putty-and-netrw)


