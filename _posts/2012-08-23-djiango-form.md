---
layout: post
title: Django form 和元类
---

django.forms.Form 类的表单字段属性能够按照定义顺序排列是一个十分友好的 feature，通过查看源码可以发现原理十分简单，只是一个对 python 元类的基本应用。


元类
----
类的类就是元类。在 python 中一切都是对象，类也是对象，是内建类 type 的实例。

{% highlight python %}
  class File(object):
      separator = '/'
      def read(self, file):
          open(file).read()
  print type(File)
  # => <type 'type'>
{% endhighlight %}

定义一个类实际上等价于生成一个 type 的实例。可以简单的认为 python 在完成类的定义时在内部调用了 type

{% highlight python %}
  def read(self, file):
      open(file).read()

  file = type('File', (object,), {'separator' : '/', 'read': read })

  print dir(file)
  # => [..., 'separator', 'read']

  # 注意类名是传给 type 的第一个参数，file 只是一个引用 File 类的变量。
  print file.__name__
  # => 'File'
{% endhighlight %}

一个继承自 type 的类，可以用于生成其他类，并在类初始化前进行一些处理。如下面的例子在类初始化前添加了一个 meta 属性。
{% highlight python %}
  class Metaclass(type):
      def __new__(cls, name, bases, attrs):
          attrs['meta'] = 'metaclass'
          return super(Metaclass, cls).__new__(cls, name, bases, attrs)

  class Usage(object):
      __metaclass__ = Metaclass

  print Usage.meta
  # => 'metaclass'
{% endhighlight %}

实际应用中可能需要通过继承一个元类的实例来隐藏元类的实现细节

{% highlight python %}
  class Use(Usage):
    pass
{% endhighlight %}

Django 中包含一个实用的函数来简化这个过程

{% highlight python %}
  def with_metaclass(meta, base=object):
      return meta("NewBase", (base,), {})

  class Use(with_metaclass(Metaclass)):
      pass
{% endhighlight %}


根据定义顺序排序
----------------

了解了元类之后，接下来的事情就简单了。定义一个 Field 类，每产生一个实例时增加一个计数器，然后在 Form 的元类中对属性根据 Filed 实例产生的顺序排序并存储到 `base_fields` 属性中。为了不让实例影响到类变量，在 Form 的构造函数中深拷贝 `base_fields`
为 `fields` 供实例调用。以下代码意为展示基本原理，具体实现还应该包括有序的字典和属性存储器等，详细内容可查看 django 源码

{% highlight python %}
  import copy

  class Field(object):
      creation_counter = 0
      def __init__(self):
          self.creation_counter = Field.creation_counter
          Field.creation_counter += 1

  class FormMeta(type):
      def __new__(cls, name, bases, attrs):
          base_fields = [(name, value) for name, value in attrs.iteritems() if isinstance(value, Field)]
          base_fields.sort(key=lambda x: x[1].creation_counter)
          attrs['base_fields'] = [name for name, value in base_fields]
          return super(FormMeta, cls).__new__(cls, name, bases, attrs)

  class Form(object):
      __metaclass__ = FormMeta

      def __init__(self):
          self.fields = copy.deepcopy(self.base_fields)
      

  class Person(Form):
      name = Field()
      barthday = Field()

  print Person().fields
  # => ['name', 'barthday']

{% endhighlight %}
