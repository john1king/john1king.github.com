---
layout: post
title: CentOS 下安装 Django
---

缓存目录设置
------------

本文中的环境全部是在 Windows 下的 CentOS 虚拟机中安装。很多操作反复执行了多次，因此先介绍一下设置缓存目录的方法，可以节省很多下载时间。

### 共享目录的设置

和宿主机器共享文件目录，虽然可以使用一些虚拟机软件提供的安装包。不过又大又不方便。更简单的办法是在宿主机设置共享目录，然后挂载到虚拟机系统下。确保有一个对共享目录具备读写访问权限的用户，在 CentOS 中执行 

{% highlight html %}
  mkdir /mnt/share/
  mount -t cifs //192.168.1.111/share /mnt/share/ -o username=yourusername,password=yourpassword
{% endhighlight %}

第一个参数是宿主机器共享目录的位置，第二个参数是挂载的目标位置，必须时一个已存在的目录。mount 的好处是只要执行一条命令就可以在所有虚拟机中像使用本地文件一般的操作共享文件。当想要卸载已 mount 的目录，可以使用 umount 命令

{% highlight html %}
  umount -l //192.168.1.111/share
{% endhighlight %}

选项 `-l` 参数让 umount 程序自动在文件没有被使用时卸载，避免 umount 过程中的长时间等待。umount 参数可以是 mount 命令的两个参数之一，如果忘记，可以使用不带参数和选项的 mount 命令查看


### 修改 yum 包缓存目录

在 CentOS 下通常使用 yum 命令安装，默认配置下在安装完成之后会移除下载的 rpm 包。需要修改配置文件如下

{% highlight html %}
  vi /etc/yum.conf
  cachedir=/mnt/share/yum
  keepcache=1
{% endhighlight %}

### 设置 pip 缓存目录

pip 是最好的 python 包管理工具，通过环境变量 `PIP_DOWNLOAD_CACHE` 可以设置 pip 的缓存目录

{% highlight html %}
  vi ~/.bash_profile
  export PIP_DOWNLOAD_CACHE=/mnt/share/pip/
  source ~/.bash_profile
{% endhighlight %}

安装环境
--------

### 安装 Apache

{% highlight html %}
  yum -y install httpd
{% endhighlight %}

设置 apache 随系统启动

{% highlight html %}
  chkconfig --levels 235 httpd on
{% endhighlight %}

### 打开防火墙的 80 端口

编辑 `/etc/sysconfig/iptables`，加入如下配置

{% highlight html %}
  -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
{% endhighlight %}

重启 iptables 服务器

{% highlight html %}
  service iptables restart
{% endhighlight %}

完成这一步，已经可以使用宿主机器的浏览器访问默认的 apache 欢迎页面了

### 安装 pip

pip 和 easy_install 都依赖 python-setuptools

{% highlight html %}
  yum -y install python-setuptools 
{% endhighlight %}

安装完 python-setuptools 后可以使用 

{% highlight html %}
  easy_install pip
{% endhighlight %}

安装 pip，不过既然设置了缓存目录，还是通过安装包来安装更快

{% highlight html %}
  curl -O http://pypi.python.org/packages/source/p/pip/pip-1.0.tar.gz
  tar xvfz pip-1.0.tar.gz
  cd pip-1.0
  python setup.py install # may need to be root
{% endhighlight %}

### 安装 Django

pip 可以安装指定版本的 python 包，如下指定安装 Django 1.3.1

{% highlight html %}
  pip install Django==1.3.1
{% endhighlight %}

### 新建项目

新建一个 Django 项目。可以创建一个标准的 `Hello world` 视图来测试服务是否运行（代码略）

{% highlight html %}
  django-admin.py startproject mysite
{% endhighlight %}

接下来将使用 `mod_wsgi` 来运行 Django 项目，因此需要新建一个 wsgi 文件。参考配置如下

{% highlight python %}
  import os
  import sys

  path = os.path.dirname(os.path.realpath(__file__))
  if path not in sys.path:
      sys.path.append(path)

  os.environ['DJANGO_SETTINGS_MODULE'] = 'mysite.settings'

  import django.core.handlers.wsgi
  application = django.core.handlers.wsgi.WSGIHandler()
{% endhighlight %}


### 安装 mod\_wsgi

{% highlight html %}
  yum -y install mod_wsgi
{% endhighlight %}

### 配置虚拟机主机

通常将主机配置放到 `/etc/httpd/conf.d/` 目录下，以 `.conf` 命名文件后缀。CentOS 的 httpd 服务会自动加载该目录下的所有配置文件。下面是一个简单的配置参考

{% highlight html %}
  <VirtualHost *:80>
      WSGIScriptAlias / var/www/mysite/django.wsgi
      <Directory /var/www/mysite>
          Order allow,deny
          Allow from all
      </Directory>
  </VirtualHost>
{% endhighlight %}

如果编写了 `Hello world` 视图，现在打开浏览器就可以看到输出了


总结
----

1. yum 命令安装软件包虽然方便，但版本都比旧
2. 本文中讲到的几乎是 apache 运行 django 项目的最小配置，而并非最佳时间。还有很多内容都没有提及，如数据库和静态文件。
3. 文中所有命令都是直接使用 root 用户执行，未涉及用户权限问题


参考链接
--------

- [How do I install from a local cache with pip?](http://stackoverflow.com/questions/4806448/how-do-i-install-from-a-local-cache-with-pip)
- [Installation instructions &mdash; pip 1.2.1.post1 documentation](http://www.pip-installer.org/en/latest/installing.html)
- [How to install Apache Server on CentOS RedHat Linux, How to configure Apacahe Server on CentOS RedHat Linux](http://dev.antoinesolutions.com/apache-server)
