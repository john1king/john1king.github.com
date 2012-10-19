---
layout: post
title: 一个常见的 python 闭包错误
---

注：本文讨论的内容仅适用于 python 2.x

## 问题

编写带参数的 decorator 时就需要用到嵌套的函数，为此遇到了个“奇怪”的问题。

{% highlight python %}
  def outer(param=None):
      def inner(foo):
          # 没有参数时使用函数的名称
          if param is not None:
              param = foo.func_name
          # do something
      return inner
{% endhighlight %}

这段代码无法正常运行。为了找出失败的原因，先看看下面的这段代码

{% highlight python %}
  bar = 'global'
  def foo():
      if False:
          bar = 'local' #1
      print bar
  foo()
{% endhighlight %}

虽然 `#1` 的赋值语句没有运行，但还是抛出了 `UnboundLocalError` 异常。可见 **python 中的变量并不是在运行时动态分配的**，这有点像在 javascript 的函数内部使用 `var` 显示声明变量时的行为。Python 无需使用变量声明，大概是在代码解析阶段帮助我们完成了这个工作。

{% highlight javascript %}
  (function(){
    var x = 1
    (function(){
      if(false){ 
        var x = 2
      }
      console.log(x)
      // => undefined
    })()
  })()
{% endhighlight %}
## 解决方案

简单的访问上级作用域的变量，只要使用一个新的变量名即可

{% highlight python %}
  def outer(var=None):
      def inner():
          local_var = var
          if local_var is None:
              local_var = 'sample'
      inner()
{% endhighlight %}

虽然无法对上级作用域的变量重新赋值，但仍然可以借助引用对象来达到类似的效果。以下摘录两个 stackoverflow 上比较典型的解决方案

{% highlight python %}
  def outer():
      def ns(): pass
      ns.x = 3
      def inner():
          ns.x += 1
      return inner

  def outer(v):
      def inner(varname = 'v', scope = locals()):
          scope[varname] += 1
          return scope[varname]
      return inner
{% endhighlight %}

参考链接
--------
- [nonlocal keyword in Python 2.x](http://stackoverflow.com/questions/3190706/nonlocal-keyword-in-python-2-x)
- [What limitations have closures in Python compared to language X closures?](http://stackoverflow.com/questions/141642/what-limitations-have-closures-in-python-compared-to-language-x-closures)
