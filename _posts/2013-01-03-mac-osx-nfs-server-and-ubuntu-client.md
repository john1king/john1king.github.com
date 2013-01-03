---
layout: post
title: 使用 Ubuntu 挂载 Mac OSX 下的 NFS 服务
---


本文环境: Ubuntu Server 12.10 虚拟机与 Mac OSX 10.74

## 启动 Max OSX 的 NFS 服务器

1) 编辑或创建 `/etc/exports` 文件，添加导出的目录

{% highlight html %}
  /nfs -alldirs -mapall=user -network 192.168.0.0 -mask 255.255.255.0
{% endhighlight %}

`-mapall=user` 该选项将客户端上的用户映射为同一个服务器用户，客户端没有权限访问被挂载的目录时（通常由客户端和服务器的用户uid不同导致）使用
`-alldirs` 选项允许客户端挂载子目录


2) 开启 `nfsd` 服务

{% highlight html %}
  sudo nfsd enable
{% endhighlight %}

每次修改配置后需要执行 `update` 命令才能生效

{% highlight html %}
  sudo nfsd update
{% endhighlight %}

3) 查看是否已成功导出目录

{% highlight html %}
  showmount -e
{% endhighlight %}


## 挂载 NFS 目录

Ubuntu 系统需要安装 `nfs-common` 才能挂载 NFS

{% highlight html %}
  sudo apt-get install nfs-common
{% endhighlight %}

查看能否连接上 NFS 服务器

{% highlight html %}
  showmount -e <server-ip-address>
{% endhighlight %}

挂载到指定位置，挂载点需要是已存在的目录

{% highlight html %}
  sudo mount -t nfs <server-ip-address>:/nfs /mnt/nfs
{% endhighlight %}

在系统启动时自动挂载则需要修改 `/etc/fstab` 文件，添加如下配置

{% highlight html %}
  <servier-ip-address>:/nfs    /mnt/nfs    nfs    rsize=8192,wsize=8192,timeo=14,intr    0    0
{% endhighlight %}

修改完 `/etc/fstab` 文件之后可以使用 `sudo mount /mnt/nfs` 来确认配置是否正确。最后重启系统即可实现自动挂载

{% highlight html %}
  sudo reboot
{% endhighlight %}

## 参考链接

- [How to Share Directories over NFS with Mac OS X](http://www.behanna.org/osx/nfs/howto1.html)
- [exports(5) OS X Manual Page](https://developer.apple.com/library/mac/#documentation/Darwin/Reference/ManPages/man5/exports.5.html) (`/etc/exports`文件格式参考)
- [Mounting NFS File Systems using /etc/fstab](http://www.centos.org/docs/5/html/5.1/Deployment_Guide/s2-nfs-fstab.html)
