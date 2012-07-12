---
layout: post
title: Ruby文件操作
---

文件操作总是最为基本和常用的，本文主要介绍 Ruby 中一些常用的文件操作 api 和陷阱

### File.expand_path

将相对路径转换为绝对路径，能够识别以 ~ 开头的用户路径

{% highlight ruby %}
  File.expand_path('~/.vim')
  # mac  => /Users/john1king/.vim 
  # win7 => C:/Users/john1king/.vim 
{% endhighlight %}

除此之外还可以用于转换 Windows 路径中的斜杠 

{% highlight ruby %}
  File.expand_path('C:\Windows\System32\cmd.exe')
  # => C:/Windows/System32/cmd.exe
{% endhighlight %}


### FileUtils

FileUtils 标准库包含很多 shell like 的文件操作方法，在 rake 中就混入了此模块。

mkdir_p 一次性建立多级路径，如果目标文件夹已存在则什么都不做

{% highlight ruby %}
  FileUtils.mkdir_p('one/two/three') 
{% endhighlight %}

cp_r 复制文件夹到目标文件夹下，将文件夹 src 复制到 target/src 时, 最好使用 target/ 做为目标路径

{% highlight ruby %}
  FileUtils.cp_r('src/', 'target/src/') 
  # target/src 不存在时，将被复制到 target/src
  # target/src 已存在时，将被复制到 target/src/src
{% endhighlight %}

## 常见陷阱

### Dir[]/Dir.glob

Dir.glob 是遍历文件夹最简单快捷的方法。但在处理命名不太规范的文件夹时，还是得留个心眼

{% highlight ruby %}
  # Files: [ruby]/glob.rb, r/glob.rb
  Dir[File.join('[ruby]/*.rb')]
  # => ["r/glob.rb"]
{% endhighlight %}

文件名中包含了 '[]' 元字符，因此没有得到预期的结果。一个简单的解决办法是进入目标文件夹后再使用 Dir[] 方法

{% highlight ruby %}
  path = '[ruby]'
  Dir.chdir(path) do 
    Dir['*.rb'].map {|f| File.join(path, f)}
  end
  # => ["[ruby]/glob.rb"]
{% endhighlight %}

### 文件名编码

在 Windows 下编码是个永远的痛苦。Ruby 1.9.3 已经能够比较好的转换文件名编码，不过还是存在一些陷阱

{% highlight ruby %}
  # Version: ruby 1.9.3p194

  # File: Ruby文件操作.md
  Dir["*.md"].first.encoding
  # => #<Encoding:GB2312>

  # Directory: 文件夹/
  File.expand_path('Rakefile').encoding
  # => #<Encoding:GBK>
{% endhighlight %}

ascii 编码的字符串与 gbk 相兼容，因此不会发生编码转换。可以通过显示的指定 utf-8 字符编码来解决

{% highlight ruby %}
  Dir["*.md".encode("utf-8")].first.encoding
  # => #<Encoding:UTF-8>
{% endhighlight %}
