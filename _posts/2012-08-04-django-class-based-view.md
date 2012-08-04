---
layout: post
title: Django 的视图类
---

在一个典型的请求和响应过程中，Django 接受到请求后调用路由中与 url 相匹配的视图函数，最后返回一个响应交给服务器传回客户端浏览器。视图类（class-based-view）本质上和视图函数没有任何不同，首先看看最基本的用法

{% highlight python %}
  # urls.py
  from my_app.views import IndexView
  urlpatterns = patterns('',
      url(r'^$', IndexView.as_views()))

  # views.py
  from django.views.generic import TemplateView

  class IndexView(TemplateView):
      template_name = "index.html"

      def get(self, request):
          return self.render_to_respone({"message" : "Hello world!"}) 
{% endhighlight %}

`as_view` 方法实际上返回了一个函数，该函数根据 http 请求类型调用 IndexView 相应的方法：get 请求对应 get 方法，post 请求对应 post 方法。 如果方法没有定义，将会返回 HttpResponseNotAllowed 对象。通过设置视图类的 `http_method_names` 属性也可以限制类视图能够接受的请求类型

{% highlight python %}
  # 只接受 post 请求的视图类
  class IndexView(TemplateView):
      http_method_names = ['post']
{% endhighlight %}

所有的视图类属性都可以中由 `as_view` 重新指定， 当视图类中不包含指定的属性时，将会抛出异常。

{% highlight python %}
  url('^$', IndexView.as_view(template_name='other.html', http_method_names=['get']))
{% endhighlight %}
  
在 TemplateView 类中已经定义了简单的get 方法

{% highlight python %}
  # django/views/generic/base.py

  def get(self, request, *args, **kwargs):
      context = self.get_context_data(params=kwargs)
      return self.render_to_response(context)
{% endhighlight %}

习惯上将模板数据的处理封装在 `get_contxt_data` 方法中，`render_to_resopnse` 负责调用模板并返回响应。 继承而来的 `get_context_data` 方法实际上只返回传递给它的关键字参数，因此重写时并不需要特意地调用父类的方法。

TemplateView 必须需指定一个模板，`render_to_response` 还方法需要一个 context 参数，在一些场景下并不是很好用。这时可以考虑直接继承 django.views.generic.View 类

{% highlight python %}
  from django import http
  from django.views.generic import View

  class IndexView(View):
      def get(self, request):
          return http.HttpResponse('Hello world!')
{% endhighlight %}

除了区分不同的Http请求外，使用方式和普通的视图函数并没有太大的差别。

Django 里可以通过视图函数来生成对应的url，在使用视图类时，并不能够直接通过视图类来获取 url。可以通过 url 的 name
参数来简单的解决这个问题

{% highlight python %}
  # urls.py
  url('^$', IndexView.as_view(), name='index')

  # views.py
  reverse('index')
{% endhighlight %}

还可以在定义视图类的模块中直接引用 `as_view` 方法返回的函数

{% highlight python %}
  # urls.py
  url('^$', 'my_app.views.index', name='home')

  # views.py
  index = IndexView.as_view()
  reverse('my_app.views.index')
{% endhighlight %}

这样看起来就和使用普通的视图函数一样了
