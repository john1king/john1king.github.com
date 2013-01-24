---
layout: post
title: Unicdoe 字符集与 UTF-8 编码
---


Unicode 规定了所有字符所对应的唯一编码（类似于ID号），最新的 Unicode 6.2 中定义了 ***110, 117* 个字符。在大部分语言中，Unicode字符串都能够以 `\uhhhh` 的形式表示。任何数据保存到计算机中都会以字节来表示。将 Unicode 字符保存为字节有很多种实现方式。比较常见的就有 UTF-32 和 UTF-8。前者以每个字符固定4个字节的方式保存，后者是可变长的编码，最大长度为6个字节。两者实际都能够储存 **2, 147, 483, 648** 个字符，在遭遇地外文明之前完全够用了。

文本主要介绍 Unicode 和 UTF-8 间的转换，关于 UTF-8 的具体实现方式可查看注释。这里仅举个简单的例子

Unicode 中 '汉' 字的二进制表示
{% highlight html %}
  1101100 01001001
{% endhighlight %}

UTF-8 中 '汉' 字的二进制表示，连接 `|` 后的二进制编码得到就是 Unicdoe 码点
{% highlight html %}
  1110|0110 10|110001 10|001001
{% endhighlight %}



## Unicode 与 UTF-8 在各个语言中的转换

介绍在 python, ruby 和 javascript 中两者间的相互转换

  1. 计算UTF-8字符串的字符长度
  2. 将UTF-8 编码的字符串转换为 Unicode 码点
  3. Unicode 码点转换为 UTF-8 编码的字节
  4. UTF-8 字符串的字节码


### Python


{% highlight python %}
  #1
  len(u'汉'.encode('utf-8')) # => 3

  #2
  ord(u'汉')  # => 27721

  #3
  unichr(22721) #=> u'汉'

  #4
  map(ord, u'汉'.encode('utf-8')) # => [230, 177, 137]
{% endhighlight %}


### Ruby

{% highlight ruby %}
  #1
  '汉'.bytesize # => 3


  #2
  '汉'.ord  # => 27721

  #3
  27721.chr('utf-8')  # => '汉'

  #4
  '汉'.bytes.to_a # => [230, 177, 137]
{% endhighlight %}


### JavaScript

在浏览器中计算 UTF-8 字节长度也十分容易，关于使用 javascript 实现的 utf-8 编码请参考本文末的链接

{% highlight javascript %}
  encodeURIComponent(text).replace(/%[A-F\d]{2}/g, 'U').length // => 3
{% endhighlight %}

## 参考链接

- [UTF-8](http://www.unicode.org/resources/utf8.html) 包含了 UCS4 与 UTF-8 相互转换的标准 C 语言实现
- [UTF-8 and Unicode FAQ for Unix/Linux](http://www.cl.cam.ac.uk/~mgk25/unicode.html)
- [Using UTF-8 with JavaScript](http://book.soundonair.ru/web/web2apps-CHP-4-SECT-10.html) 在 Javascript 转换 Unicode 字符为 UTF-8 字节码
- [字符编码笔记：ASCII，Unicode和UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)
- [Count bytes in textarea using javascript](http://stackoverflow.com/questions/2848462/count-bytes-in-textarea-using-javascript)
