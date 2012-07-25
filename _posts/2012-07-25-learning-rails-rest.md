---
layout: post
title: 认识 Rails REST
---


REST(表述性状态转移，Representational State Transfer) 架构的应用程序将一个URL视作一个资源，使用 HTTP 的 GET, POST, PUT, DELETE 方法来访问，创建，更新和删除资源。 Rails 将 REST 组织成更适合 Web 应用程序的形式, 使得构建路由和 URL 变得异常简单。

首先在 routes.rb 中添加一个资源路由

{% highlight ruby %}
  # config/routes.rb
  resources :products
{% endhighlight %}

使用 rake routes 可以方便的查看所生成的路由和对应的方法

{% highlight html %}
      products GET    /products(.:format)          products#index
               POST   /products(.:format)          products#create
   new_product GET    /products/new(.:format)      products#new
  edit_product GET    /products/:id/edit(.:format) products#edit
       product GET    /products/:id(.:format)      products#show
               PUT    /products/:id(.:format)      products#update
               DELETE /products/:id(.:format)      products#destroy
{% endhighlight %}

如果控制器的名称为单数, 那么资源集合的路由相应的变为 controller\_index

{% highlight html %}
  product_index GET    /product(.:format)          product#index
                POST   /product(.:format)          product#create
{% endhighlight %}

每个方法对应一个资源，因此一个 resources 路由总共产生4个资源: 产品的集合，单个产品，添加产品时的表单，编辑产品时的表单。 其中后两者属于用户界面, 负责将用户信息提交到 create 和 update 这两个动作,在一些只提供API的应用程序中不是必须的，可以在定义路由的时禁用它们。

{% highlight ruby %}
  resources :products, :except => [:new, :edit]
{% endhighlight %}

创建新产品是向 "/products" 发送 POST 请求，这个动作可以理解为向集合添加新的资源，因为需要创建的资源还不存在，只能指明将它添加到哪里。


在视图和控制器中，资源路由生成的4个方法名加上 _url 或 _path 后缀的方法可以用于构建资源的 URL地址 或 路径。

{% highlight ruby %}
 products_url
 # => http://0.0.0.0:3000/products

 products_path
 # => /products

{% endhighlight %}

对于单个的资源，Rails 通过资源 id 来构建 URL，因此需要传递 id 或 资源的实例（包含了id属性）做为参数

{% highlight ruby %}
  product_path(1)
  # => /products/1 

  product_path(:id => 1)
  # => /products/1 

  @product = Porduct.find(1)
  product_path(@product)
  # => /products/1 
{% endhighlight %}

更进一步，在 Rails 中以 url 为参数的方法都可以接受一个实例变量作为参数来构建 url

{% highlight erb %}
  <%= link_to "Show", @product %>
{% endhighlight %}

这些方法同时还可以接受一个指定HTTP请求动作的参数

{% highlight erb %}
  <%= link_to "Show", @product, :method => :delete %>

  <!-- app/views/products/new.html.erb -->
  <%= form_for(@product, :url => products_url do |f| %>
    ...
  <% end %>

  <!-- app/views/products/eidt.html.erb -->
  <%= form_for(@product, :url => products_url(@post), :html => { :method => "put"}) do |f| %>
    ...
  <% end %>

{% endhighlight %}

由于创建新资源时返回的资源实例总是没有 id 的，因此上面的两个表单方法还可以进一部简化为

{% highlight erb %}
  <%= form_for(@product) do |f| %>
    ...
  <% end %>
{% endhighlight %}

Rails 会根据资源是否为新建的来自动生成表单提交的地址和HTTP动作，最终使得编辑和创建资源时的表单调用奇妙的保持一致。
