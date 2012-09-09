---
layout: post
title: Python tricks
---

本文的内容主要来自《Python 基本教程第二版》，摘录了比较有用但容易被忽视的小技巧。代码全部基于 Python 2.7.3


## 运算符与语句

重定向 print 输出
{% highlight python %}
  log = open('log', 'w')
  print >> log, 'message'
{% endhighlight %}


python 中没有三元操作符，只有类似的条件表达式
{% highlight python %}
  value = 1 if True else 0 
  print value
  # => 1
{% endhighlight %}

条件判断中的假值(空值)
{% highlight python %}
  False None 0 "" () [] {}
{% endhighlight %}

可以连续使用的比较运算符
{% highlight python %}
  age = 17
  print 0 < age < 18
  # => False
{% endhighlight %}

整除操作符 `//` 常用计算数组下标, 类似于 JS 中的 `Math.floor(array.length/n)`
{% highlight python %}
  lst = range(9)
  lst[:len(lst)//2]
  # => 4
{% endhighlight %}

成员检查, 注意空字符串的情况
{% highlight python %}
  print 1 in [1,2,3]
  print 'a' in {'a': 1, 'b':2}
  print '' in 'abc'
  # => True
{% endhighlight %}

## 转换说明符

使用转换说明符代替字符串连接
{% highlight python %}
  print str(1) + '+' + str(2)
  # => 1+2

  print '%d+%d' % (1, 2)
  # => 1+2
{% endhighlight %}

在转换说明符中指定字符串精度
{% highlight python %}
  print '%.2s' % 'string'
  # => st
{% endhighlight %}

使用字典作为转换说明符的参数
{% highlight python %}
  '%(name)s' % {'name': 'john' }
  # => john
{% endhighlight %}


## 函数与方法

判断字符串中是否只包含空白字符
{% highlight python %}
  ' \t\n'.isplace()
  # => True
{% endhighlight %}

字符串的 find 方法可以制定查找范围
{% highlight python %}
  print 'abc123abc'.find('abc', 3, 6) 
  # => -1
{% endhighlight %}

sort 方法默认按升序排序元素, reverse 参数可以指定排列顺序
{% highlight python %}
  lst = [1,2,3]
  lst.sort(reverse=True)
  print lst
  # => [3,2,1]
{% endhighlight %}

将二维数组(或元组)转换为字典
{% highlight python %}
  print dict([('a', 1), ('b',2)])
  # => {'a':1, 'b':2}
{% endhighlight %}

通过 dict 函数的关键字参数创建字典
{% highlight python %}
  print dict(a=1,b=2)
  # => {'a':1, 'b':2}
{% endhighlight %}

在 update 方法中使用这个技巧更为有用
{% highlight python %}
  d = {}
  d.update(a=1,b=2)
  print d
  # => {'a':1, 'b':2}
{% endhighlight %}

使用 setdefault 方法省去对字典健值的判断
{% highlight python %}
  d1 = {}
  for i in range(2):
      if 'children' not in d1:
          d1['children'] = []
      d1['children'].append(i)

  # 可改写为
  d2 = {}
  for i in range(2):
      d2.setdefault('children', []).append(i)
{% endhighlight %}


## 循环与迭代

在迭代时为序列加上序号
{% highlight python %}
  for index, value in enumerate('abc'):
      print index, value

  # => 
  # 0 a
  # 1 b
  # 2 c
{% endhighlight %}

循环的 else 语句，在没有调用循环的时候执行
{% highlight python %}
  for x in []:
      print x
  else:
      print 'not element'
  # => not element
{% endhighlight %}

列表推导式可以包含多个 for 语句
{% highlight python %}
  lst = [[1,2], [3,4]] 
  print [i for a in lst for i in a]
  # => [1,2,3,4]
{% endhighlight %}
