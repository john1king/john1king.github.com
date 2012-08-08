---
layout: post
title: Python 模块和包
---

模块和包是 python 组织的代码最重要形式。一个 `.py` 文件被视为一个模块(module)，一个包含 `__init__.py` 的文件夹被当作一个包(package)。 

## 导入

基本用法

{% highlight python %}
  import module 
  import module_a, module_b, module_c
  import module as alias
  from pageage import module
{% endhighlight %}

python 不会重复导入一个模块

{% highlight python %}
  f = open('a.py', 'w'); f.write('n = 1'); f.close()
  import a
  print a.n 
  #=> 1

  f = open('a.py', 'w'); f.write('n = 2'); f.close()
  import a as b
  print b.n
  #=> 1

  from a import n
  print n
  #=> 1
{% endhighlight %}

如果没有使用 from 或 as 语句，则需要用全名来引用导入的模块。这一点和后文中提到 `__import__` 是一致的，实际上只导入了第一个模块
{% highlight python %}
  import package.module 
  package.module.some_variable
{% endhighlight %}

在一个包中的模块，可以使用 `from .` 引用同一个包的其它模块。这时候如果直接执行包中的模块将会抛出异常

{% highlight python %}
  # current.py
  from pakage import module
  module.a
  module.c

  # package/
  #   module.py
  from . import a
  from .b import c

  # ...
{% endhighlight %}

`from` 语句的另一个好处是可以直接导入模块中的方法

{% highlight python %}
  from package.module import method
{% endhighlight %}


## 命名冲突

模块的加载路径由 sys.path 指定，默认情况下当前目录在第一位。如果当前运行的路径下包含和标准库同名的文件，使用 import 语句将只会得到当前目录下的模块。这时想要导入标准库，需要额外的 

{% highlight python %}
  def import_non_local(name, custom_name=None):
      import imp, sys

      custom_name = custom_name or name

      f, pathname, desc = imp.find_module(name, sys.path[1:])
      module = imp.load_module(custom_name, f, pathname, desc)
      f.close()

      return module

  import_non_local('json')
{% endhighlight %}

不建议模块或包使用与标准库或关键字相同的名称

## 动态导入模块

魔术方法 `__import__` 接受字符串参数并返回一个入模块对象，可以通过声明变量来引用模块。如果希望导入一个包里的模块，还必须传递 formlist 参数
，否则得到的只是最上层的包

{% highlight python %}
  m = __import__('m')

  p1 = __import__('package.module')
  # => <module 'package' ... >

  p2 = __import__('package.module', fromlist=['pageage'])
  # => <module 'package.module' ... >
{% endhighlight %}


## 包的初始化

在 `__init__.py` 文件中可以进行一些包的初始化,`__init__.py` 中的全局变量将会成为包的属性。例如希望直接在包中调用子模块定义的方法，可以

{% highlight python %}
  # package/
  #   __init__.py
  from .module import *

  #   module.py
  def hello(word):
      print "Hello " + word + '!'
    
  # current.py
  import package
  package.hello('python')  # => Hello python!

{% endhighlight %}


## 包和模块的属性

包和模块也是对象，能够使用赋值操作创建新的属性。除此之外还有一个重要的特性，模块内部的全局变量实际上引用了模块的属性。可以在模块的函数中引用未声明的全局变量，等到执行时再动态设置模块的属性。

{% highlight python %}
  # module.py
  def hello():
      print variable

  # main.py
  import module
  module.variable = 'cool!'
  module.hello()
  # => 'cool!'

{% endhighlight %}


## 参考链接

- [Why does Python's __import__ require fromlist?](http://stackoverflow.com/questions/2724260/why-does-pythons-import-require-fromlist)
- [python: importing from builtin library when module with same name exists](http://stackoverflow.com/questions/6031584/python-importing-from-builtin-library-when-module-with-same-name-exists)
