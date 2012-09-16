---
layout: post
title: 迭代
---


循环
----

循环是编程语言中最基本的部分，大量重复的计算都是在循环中完成的。如下列的 JavaScript 代码，是我们都曽重复过过无数次 for 循环，它有固定的结构，以至于大部分时间我们都在重复输入这些无趣的代码。当然优秀的编辑器和 IDE 都可以自动补全这些代码片段，但这并无助于让代码变得简洁。

{% highlight javascript %}

  var ary = [1, 2, 3]
  var map = []

  for(var i = 0, len = ary.lenth; i < len; i++){
    map[i] = ary[i] + 1
  }

  console.log(map)
  // => [2, 3, 4]

{% endhighlight %}


高阶函数
--------

高阶函数指能够将函数作为参数或返回值的函数。有了高阶函数之后，可以提取出循环的代码，然后只关注主体部分。多数动态语言如 Ruby, Python, JavaScript 等都支持高阶函数，其中 Ruby 因为有代码块,各种语法糖以及内建的大量迭代方法，使用高阶函数尤为简单，以至于几乎没有 for 循环的用武之地。

map, filter/select, reduce/inject 都是最常见的高阶函数。inject 的行为稍微复杂些，它将上一次迭代中代码块的返回值和被迭代元素一起传递给代码块

{% highlight ruby %}

  [1, 2, 3].map {|n| n + 1 }
  # => [2, 3, 4]

  [1, 2, 3].select {|n| n > 1 }
  # => [2, 3]

  [1, 2, 3].inject {|n, m| n + m }
  # => 6
{% endhighlight %}

Ruby 中的 & 操作符调用对象的 to_proc 方法，并将返回的代码块对象转换成真正的代码块参数。 如将字符串数组转换为整数数组可以简单的写为

{% highlight ruby %}
  ['1', '2', '3'].map(&:to_i)
  # => [1, 2, 3]
{% endhighlight %}

Ruby 并没有实现 sum 方法，因为借助 inject 和 & 操作符就可以轻易的实现

{% highlight ruby %}
  #ruby 中的求和与阶乘
  ary = [1,2,3,4]

  ary.inject(&:+)
  # => 10

  ary.inject(&:*)
  # => 24
{% endhighlight %}

虽然刚开始看起来有些古怪，但并不难理解而且十分易用



### Enumerable ###

map, select, inject 方法实际上来自 Enumerable 模块。任何对象只要定义了 each 方法并混入 Enumerable 模块都可以使用这几个方法

{% highlight ruby %}
  class CountDown
    include Enumerable
    def each
      3.downto(1) {|n| yield n}
    end
  end

  CountDown.new.map {|n| n - 1}
  # => [2, 1, 0]
{% endhighlight %}

Enumerable 模块实例了很多常用的方法。当这些方法没有接收到代码块参数时，会返回一个可迭代对象(Enumerator)，这个仍然能够调用  Enumerable 模块的方法。因此可以联合使用几个迭代器，一次达成目标。下面的例子为选择偶数位置的字符的一种方法


{% highlight ruby %}
  'abcdefg'.chars.select.with_index(1) {|c, i| i.even? }
  # => ['b', 'd', 'f']
{% endhighlight %}

可迭代对象能够转换成数组，下面的例子返回字符串的字节码

{% highlight ruby %}
  'abc'.bytes.to_a
  # => [97, 98, 99]
{% endhighlight %}


列表推导式
----------

Python 虽然也有 map, filter, reduce 等函数，但在语法上得到支持的确实列表推导式。列表推导式是一种轻量级的行内循环，能够以非常简洁的方式实现常见的数据处理，在 Python 中使用极为广泛。

列表推导式支持条件判断和多重循环，实现 map 和 filter 十分简单

{% highlight python %}
  ary = [0, 1, 2]
  print [n + 1 for n in ary ]
  # => [1, 2, 3]


  ary = [0, 1, 2]
  print [n for n in ary if n > 0]
  # => [1, 2]
{% endhighlight %}

可以利用多重循环来展开二维数组

{% highlight python %}
  ary = [[0, 1], [2, 3]]
  print [n for a in ary for n in a]
  # => [0, 1, 2, 3]
{% endhighlight %}

reduce 函数无法简单的使用列表推导式来替换，当然在 Python 中，这个函数使用的并不如 Ruby 中频繁。 大量的列表推导式任何降低代码的可读性，应该适度的使用和封装。


ECMAScript 5
------------

ECMAScript 5 已经实现了数组的 forEach, map, filter, reduce 等方法。对一些旧的浏览器需要使用 [es5-shim](https://github.com/kriskowal/es5-shim) 或 [es5-safe](https://github.com/seajs/modules/tree/gh-pages/es5-safe) 来兼容。

这些类库扩展了内建对象的原型，使其行为与 ES5 规范基本保持一致。在 ECMAScript 5 实现这些方法之前，很少有类库通过扩展内建对象的原型来实现这些基本的数组方法。

主流的浏览器都有自己的实现，在形成统一的规范前，浏览器厂商经常会添加一些实验性的方法。随意的扩展内建对象的原型，当浏览器实现了同名但不同行为的方法后，会带来很麻烦的维护问题。因此类似 jQuery 和 Underscore 这种不扩展原生对象的库更为流行。


jQuery
------

jQuery 的迭代器并不多，而且主要用于处理 DOM 对象，因此行为和其它类库有所不同。

$.each 方法将回调函数中的 this 指向被迭代的元素，传递数组索引为第一个参数，被迭代元素的值第二个为第二个参数。注意当被迭代元素是数字和字符串等非引用对象时，this 对象将会是一个包装类，而不是值对象。这时应该始终使用第二个参数作为值来使用

{% highlight javascript %}
  $.each([0, undefined, ''], function(index, value){
    console.log(this)
  })
  // => Number, Window, String
{% endhighlight %}


jQuery 中的 map 方法行为比较特殊, 会移除返回值中的 null 和 undefined, 并展开数组。此外回调函数的参数和 each 方法也是不同的，数组值在前，索引在后


{% highlight javascript %}
  ary = [null, 0 , undefined, [2, 1]]
  $.map(ary, function(value, index){
      return value
  })
  // => [0, 2, 1]
{% endhighlight %}

如果希望保留原始的返回值，需要用点小技巧，将返回值写在数组中

{% highlight javascript %}
  $.map(ary, function(value, index){
      return [value]
  })
  // => [null, 0 , undefined, [2, 1]]
{% endhighlight %}

Underscore
----------

Underscore 在 JavaScript 上实现了一套 Ruby Like 的 API，虽然很少有机会用到其中全部的方法，但作为一个参考实现还是十分有价值的。

Underscore 的 this 对象用法与 jQuery 有很大不同。允许传递一个参数作为回调函数的 this 对象，这在嵌套的函数作用域中能够省去引用 this 对象的临时变量

{% highlight javascript %}
  (function(){
    return _(['a', 'b', 'c']).map(function(e){
      return this.prefix + e
    }, this)
  }).call({ prefix: '_' })
  // => ["_a", "_b", "_c"]
{% endhighlight %}

具体的 API 可查看官方文档


参考
----
- [Array | Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Array)
- [扩展原生对象与 es5-safe 模块](http://lifesinger.wordpress.com/2011/08/10/extending-built-in-native-objects/)
- [Underscore.js](http://underscorejs.org/)
