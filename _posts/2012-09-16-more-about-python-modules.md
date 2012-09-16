---
layout: post
title: Python 模块的一些应用 
---


使用 Python 的这段时间，让人印象最深的当属模块。它无处不在，直接影响了程序的书写方式。模块有很多优点，例如 命名空间和严格的作用域管理。但重复的 import 令人厌倦且难以维护，因此本文探讨了一些非常规的模块使用方式。

  1. 使用模块重载为框架打补丁
  2. 相互导入模块的常见错误与解决之道
  3. 介绍简化包模块相互引用的一种方法


## 模块重载

使用模块扩展框架的一个简单技巧。下面以 Django 为例，为 From 添加一个获取 cleaned_data 的属性，在没有通过验证时直接抛出异常。

{% highlight python %}
  # forms.py
  from django.forms import *

  class Form(Form):
      @property
      def cleaned_data_force(self):
          if self.is_valid():
              return self.cleaned_data
          raise ValidationError('error')
{% endhighlight %}


如果不喜欢 `class Form(Form)` 这种看起来很奇怪的继承方式，或者有很多类需要修改，可以用包来代替模块

{% highlight python %}
  # forms/__init__.py
  from django.forms import *
  from .forms import *

  # forms/forms.py
  from django import forms

  __all__ = ['Form']

  class Form(forms.Form):
      @property
      def cleaned_data_force(self):
          if self.is_valid():
              return self.cleaned_data
          raise forms.ValidationError('error')
{% endhighlight %}

看起来要复杂一些，但代码多时这样做完全是值得的。而且两者在使用方式上并无区别， 都只要忘记框架的模块，导入上面定义的代理层即可。

{% highlight python %}
  # views.py
  from django import http
  from . import forms

  class Book(forms.Form):
      name = forms.CharField()

  def book(request):
      try:
          result = Book({}).cleaned_data_force
      except forms.ValidationError: 
          result = 'error'
      return http.HttpResponse(result)
{% endhighlight %}

成熟的框架应该尽量使用框架自身定义好扩展方式

## 相互导入

在关联的模块中，经常需要互相引用对方。虽然 Python 会记住已导入的模块，当在相互导入时还是经常遇到错误。

{% highlight python %}
  # books.py
  form authors import Author
  class Book(object):
      pass

  # authors.py
  form books import Book
  class Author(object):
      pass

{% endhighlight %}

假设先执行的是 `books.py`，那么 books 模块会模块将被执行两次，在第二次执行时，Python 认为 authors 模块已经被导入，而实际上 authors 模块中的大部分代码还未执行，Author 也没有被定义。因此 `form authors import Author` 出错。

避免出现这类错误的最好方法是使用良好的设计模块，避免模块间的相互引用，大部分相互引用的模块都是可以被拆解的。如果因为复杂的需求问题而使用这种结构，可以在方法或函数中导入模块

{% highlight python %}

  # books.py
  class Book(object):
      def authors(self):
          form authors import Author

  # authors.py
  class Author(object):
      def books(self):
          form books import Book
{% endhighlight %}

虽然绕过了错误，但方法和模块较多时仍会带来很多的重复代码。由于在方法或函数中的模块只有执行的时候才会被调用，可以动态的获取这些模块。

{% highlight python %}
  # base.py
  import sys
  class Package(object):
      def __init__(self, path=''):
          self.path = path 

      def __getattr__(self, name):
          if self.path:
              name = '%s.%s' % (self.path, name)
          try:
            return sys.modules[name]
          except KeyError:
            return __import__(name, fromlist=[self.path])

  package = Package()

  # books.py
  from base import package
  class Book(object):
      def authors(self):
          package.authors.Author

  # authors.py 
  from base import package
  class Author(object):
      def books(self):
          package.books.Book
{% endhighlight %}

## 约束

规范模块间相互引用仍然是有必要的，可以减少不必要的麻烦。

  1. 在包的 __init__.py 中导入所有模块的公共方法
  2. 在 base.py 中定义超类和被其它模块依赖的方法，以包名来命名导入器的名称
  3. 在模块的 __all__ 属性中定义公共方法，并使用 package.interface 访问其它模块

下面是一个简单的实现

{% highlight python %}
  # models/__init__.py
  from .books import *
  from .authors import *

  # models/base.py
  import sys
  __all__ = ['models', 'Base']
  class Package(object):
      def __init__(self, package):
          self.package = sys.modules[package] 
      def __getattr__(self, name):
          return getattr(self.package, name)
  class Base(object):
      pass
  models = Package('models')

  # models/books.py
  from .base import *
  __all__ = ['Book']
  def book_helper(): 
      pass
  class Book(Base):
    def authors(self):
        models.Author()

  # models/authors.py 
  from .base import *
  __all__ = ['Author']
  class Author(Base):
    def books(self):
        models.Book()
{% endhighlight %}

最后，不要在错误的路上走的太远。
