---
title: hexo评论框部署
date: 2016-05-23
categories: hexo
tags:
  - disqus
  - duoshuo
---
这几天我被一个问题缠了很久，一直没解决，到现在算是有点清楚了，所以来写个博文记录一下。hexo本身是有默认的评论框的disqus，个人感觉这个评论框风格很好，其实我想用来着，不过想想国内要翻墙，很麻烦，所以很委屈的选择了多说，不是说多说不好，自己个人比较喜欢disqus。废话不多说，来看看我折腾的过程吧。

<!--more-->

#### 配置disqus评论框
我强烈推荐有条件的人选择disqus，而且由于是默认的评论框，很简单，你先去注册一个disqus账号，然后获取你的disqus_shortname,接着你要找到你的博客主目录下的_config.yml文件，这个文件是设置你的整个博客的配置的。然后找到如下代码

``` bash
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: landscape

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
```

在theme： landscape下面添加如下代码

``` bash
# Disqus
disqus_shortname: your short_name
```

然后保存就可以了，如果不行，在网上搜一下吧，反正我的是可以的

#### 配置duoshuo评论框
这个可能对于一点没接触过html，javascript和markdown的人来说是很痛苦的事情，因为你看不懂代码，然后网上有些博主写得很简洁，就看不懂了，我也是，所以我就一直抱着看呀看，看呀看，觉果懂了那么一点。
首先也是要去多说网站注册一个账号，然后获取你的short_name,然后还是在上面说的那个文件中将刚刚要插入的代码改成

``` bash
# Duoshuo
duoshuo_shortname: your short_name
```

然后找到这个文件：themes/你的博客所用主题/layout/_partial/article.ejs, 打开它。然后你可以疯了，因为当时我很难看懂里面代码什么意思，不过别放弃，慢慢找代码，找到

``` bash
<footer>
....
....
</footer>
```
这个小框架，然后不管你看到了什么，都把它改成下列代码

``` bash
<footer class="article-footer">
      <a data-url="<%- post.permalink %>" data-id="<%= post._id %>" class="article-share-link"><%= __('share') %></a>
      <% if (post.comments && theme.duoshuo_shortname){ %>
        <a href="<%- post.permalink %>#disqus_thread" class="article-comment-link"><%= __('comment') %></a>
      <% } %>
      <%- partial('post/tag') %>
    </footer>
```

然后在这个文件最后将你在多说网站上获取的代码复制粘贴好，不过没完，你需要完善，完善什么呢，你会看到其中有些值需要你去填写。这个过程可以在网上找一下，因为看不懂，如果你自己打代码，会出现一些小细节的语法错误，最后完善了就像下面一样

``` bash
<% if (!index && post.comments && theme.duoshuo_shortname){ %>
<section id="comment">
<div class="ds-thread" data-thread-key=<%= page.path %> data-title=<%= page.title %> data-url=<%= page.permalink %>></div>
<!-- 多说评论框 end -->
<!-- 多说公共JS代码 start (一个网页只需插入一次) -->
<script type="text/javascript">
var duoshuoQuery = {short_name:"your shortnsme"};
	(function() {
		var ds = document.createElement('script');
		ds.type = 'text/javascript';ds.async = true;
		ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
		ds.charset = 'UTF-8';
		(document.getElementsByTagName('head')[0]
		 || document.getElementsByTagName('body')[0]).appendChild(ds);
	})();
	</script>
    </section>
    <%}%>
```

随便解释一下，第一行是条件语句，大概意思是主页上的文章不显示评论，要点击评论后才显示评论，接下来的代码就是评论框的实现方法。修改完了，现在可以保存了，然后部署到你的github page上，如果你能看到效果，没有什么问题，那么恭喜你，你太幸运了。反正对于一开始连代码都看不懂的我来说出现了好多问题，现在想来都觉得，明明很简单的事情，我却踩了好多坑。

#### 附文
之前说的那个comment小bug让我知道了另外一个东西，就是每篇博文下的comment,share是怎么实现的。下附网址：
https://astronautweb.co/snippet/font-awesome/
怎么使用别问我，自己慢慢摸索吧，着个人感觉是个好东西。
