---
layout: post
title: Django 多语言
---

安装环境
--------

1. Windows 和 Mac OSX 需要安装 `xgettext` 才能够生成语言包  
  **Windows**  
  [https://docs.djangoproject.com/en/dev/topics/i18n/translation/#gettext-on-windows](https://docs.djangoproject.com/en/dev/topics/i18n/translation/#gettext-on-windows)  
  **Mac OSX**  
  [http://gpiot.com/django-internationalization-on-osx-lion/](http://gpiot.com/django-internationalization-on-osx-lion/])

2. 在项目或 app 根目录下创建 `locale` 文件夹。这是 django 默认保存语言包的位置，可以结合 `django-adming.py makemessages -l` 和配置文件的 `LOCALE_PATHS ` 选项修改

配置文件
--------

- `USE_I18N`: 默认为 `True`，打开多语言支持

- `MIDDLEWARE_CLASSES`: 添加 `django.middleware.locale.LocaleMiddleware` 到此选项

- `INATALLED_APPS`: 添加 app 到此选项中，django 只会搜索已安装的 app 目录下的 `locale` 文件夹

- `LANGUAGES`: 支持的语言列表

- `LANGUAGE_CODE`: 修改默认语言

- `LOCALE_PATHS` : 自定义语言包的位置，优先级比 `locale` 文件夹高

生成和编译语言包
----------------

在 项目/app 根目录下执行生成语言包的命令

{% highlight html %}
  django-admin.py makemessages -l zh_CN
{% endhighlight %}

注: `zh_CN` 部分使用的是`语言代码_国家代码`格式的**本地名称(locale name)**，和 django 配置文件中使用的**语言名称(language name)** 是不同的

更新已生成的语言包

{% highlight html %}
  django-admin.py makemessages -a
{% endhighlight %}

生成的语言包需要编译之后才能够使用，执行

{% highlight html %}
  django-admin.py compilemessages
{% endhighlight %}


翻译文本
--------

在视图中使用 `gettext` 翻译文本，惯例上使用 `_` 作为缩写

{% highlight python %}
  from django.utils.translation import gettext as _

  def home(request):
      output = _('wtf')
      return HttpResponse(output)
{% endhighlight %}

`makemessages` 命令是以文本匹配的方式找到所有作为 `gettext()` 或 `_()` 中参数的字符串。 django 在每次生成语言包时，都会清理掉没有被使用的字符串。因此通过变量来翻译文本时，最好不要直接在语言包中添加，而是
定义一个返回字符串自身的 `gettext` 方法。

{% highlight python %}
  # sample_setting.py
  gettext = lambda s: s
  AUTHOR = gettext('my name')

  # view.py
  from django.utils.translation import gettext as _
  from project.path.sample_setting import AUTHOR

  def home(request):
      return HttpResponse(_(AUTHOR))
{% endhighlight %}

设置语言和访问
--------------

django 内部已经实现了设置语言的视图。需要做的只是在配置一个新的 url。 设置语言时向`/i18n/setlang/` 提交 POST 请求，目标语言的参数为 `lanuage`

{% highlight python %}
    urlpatterns += (r'^i18n/', include('django.conf.urls.i18n'), )
{% endhighlight %}

url 前缀是另一种常见的使用多语言的方式

{% highlight python %}
  from django.conf.urls.i18n import i18n_patterns

  # ...

  urlpatterns += i18n_patterns('',
      url(r'^', include(urlpatterns)),
  )
{% endhighlight %}

参考文档
--------
- [Translation | Django documentation | Django](https://docs.djangoproject.com/en/dev/topics/i18n/translation/)
