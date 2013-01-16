---
layout: post
title: Apache 中使用链接文件
---


## 链接文件

{% highlight html %}
  ln -s /mnt/share/web/ /var/www/web
{% endhighlight %}

删除链接文件

{% highlight html %}
  rm -f /var/www/web
{% endhighlight %}

注意链接名称末尾不能有 `/`


## 关闭 SELinux

如果链接的文件无法使用，并且日志文件输出 `Symbolic link not allowed or link target not accessible` 错误，则需要关闭 SELinux

临时关闭，无需重启

{% highlight html %}
  setenforce 0
{% endhighlight %}

永久关闭需要修改配置文件并重启

{% highlight html %}
  # vi /etc/selinux/config
  SELINUX=disable
  # ...

  sudo reboot
{% endhighlight %}

## Apache 配置

默认配置下 apache 启用了 `FollowSymLinks` 选项，可以添加如下配置关闭该选项

{% highlight html %}
    <Directory "/">
        Options -FollowSymLinks
    </Directory>
{% endhighlight %}
