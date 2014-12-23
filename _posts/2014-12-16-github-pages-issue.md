---
layout: post
title: GitHub Pages Issue
category: GitHub Pages
keywords: github pages, issue
description: 
---

## 1. if the blog is forked from other, then must update and commit at least one time before it works with GitHub.


## 2. if not, had better open the blog link from GitHub sites, rather than direct link.


## 3. GitHub Pages directory: _site

It's missing, and don't know why ?


## 4. should install bellows:

<!--more-->

- [RubyInstaller Development Kit](https://github.com/oneclick/rubyinstaller/wiki/Development-Kit)   
- [DevKit Overview](http://rubyinstaller.org/add-ons/devkit)


## 5. Liquid Exception: cannot load such file -- yajl/2.0/yajl

windows下如果以pygments.rb为高亮plugin，并使用Ruby 2.0及以上版本，会出现如下错误：

![yajl load error]({{ site.picture_dir }}/github-pages-issue/yajl_load_error.gif "cannot load yajl")


原因是install的预编译 yajl-ruby gems `x86-mingw32` 无法兼容 Ruby 2.0   
下面是 [RubyInstaller announcement for Ruby 2.0.0 release][RubyInstaller 2.0]:

```
* Existing pre-compiled gems are not Ruby 2.0 compatible 

Ruby 2.0 introduces ABI breakage which means compiled C extensions with previous 
1.9.3 will work with Ruby 2.0. 

DO NOT install Ruby 2.0 on top of existing Ruby 1.9.3, or try to use compiled 
extensions with it. 

You will be required to force compilation of those gems: 

    gem install <name> --platform=ruby 

This will require you have the extra dependencies required for that gem to 
compile. Look at the gem documentation for the requirements. 
```

解决方法，要么改用1.9.x的 Ruby ，要么下载要求的[.gem][yajl]文件(not the pre-compiled one)，重新安装`yajl`

```sh
gem uninstall yajl-ruby
# 删除存在的问题 gems (x86-mingw32)
gem install yajl-ruby -v 1.1.0 --platform=ruby
# 安装要求的gem文件(必需指定version，pygments.rb 0.6.0 只依赖 yajl-ruby 1.1.0)
bundle check
```

__Note:__   
Bundler will keep attempting to install x86-mingw32, so you will need to be careful when doing `bundle install` or `bundle update`.

或者也可以试试下载[.gem][yajl]文件，本地编译，不过windows下面可能不行，因为没有`Make`命令。

```sh
# build it yourself by installing rubyinstaller, the devkit and running:
gem install gem-compiler
gem fetch yajl-ruby --platform=ruby -v 1.1.0
gem compile yajl-ruby-1.1.0.gem

# to use the binary gem:
gem uninstall yajl-ruby
gem install yajl-ruby-1.1.0.gem
```

相关的 yajl-ruby issue:   
- [brianmario/yajl-ruby#116](https://github.com/brianmario/yajl-ruby/issues/116)   
- [jekyll/jekyll-help#50](https://github.com/jekyll/jekyll-help/issues/50)

[RubyInstaller 2.0]: https://groups.google.com/d/msg/rubyinstaller/mg5ailNICvM/QbBfNByec-0J
[yajl]: http://rubygems.org/gems/yajl-ruby/versions/1.1.0


## 6. Liquid Exception: No such file or directory - python C:/Ruby193/lib/ruby/gems/1.9.1/gems/pygments.rb-0.5.0/lib/pygments/mentos.py

Check your `PATH` environment variable, like `;C:\python27`;   
Make sure that have installed Python 2.x and set env path for pygments.rb call.

相关的 Jekyll issue:   
- [jekyll/jekyll#2551](https://github.com/jekyll/jekyll/issues/2551)

__Note:__   
之所以要安裝 Python 是因為 代码高亮 plugin -- [pygments.rb][]，是基于 Python 的代码高亮工具 [Pygments][] 的一个 Ruby wrapper，雖然 Pygments 支援 Python 2 版和 3 版，不過由於 Ruby 和 Python 之間的橋接是用 RubyPython 完成，而 [rubypython][] 目前只支援 Python 2。所以還是乖乖安裝 2 版吧！（见pygments.rb的`README.md`）

[pygments.rb]: https://github.com/tmm1/pygments.rb
[Pygments]: http://pygments.org/
[rubypython]: http://www.rubydoc.info/gems/rubypython/0.6.3/frames#Requirements


## 7. Jekyll 强制 Categories 单词小写

`post`可以定义categories和tags，但是Jekyll解析post时会自行将`categories`转成小写单词输出，而`tags`不会转换。   
比如，post的YAML front matter 是：

```yaml
---
layout: post
category: Mathematica//Math-Experiment
Tags: Formula Periodic Sequence
---
```

而Jekyll会将category转换为`mathematica//math-experiment`输出。   
如果不想Jekyll输出小写category，可以变通下让单词首字母大写显示，但无法还原post定义的样式。   
而如果GitHub上面deploy的是`_site`文件，那么可以local更改源码post.rb文件让其输出最初定义的格式，不过这样可能有潜在的问题(可以规避)。具体见下面的issue链接。

{% raw %}
<pre><code>{% for tag in page.categories %}
&lt;a href="{{ site.url }}/categories/index.html#{{ page.categories | cgi_encode }}" data-toggle="tooltip" title="Other posts from the {{ <font color="red">tag | capitalize</font> }} category" rel="tag"&gt;{{ <font color="red">tag | capitalize</font> }}&lt;/a&gt;
{% unless forloop.last %}&amp;nbsp;&amp;bull;&amp;nbsp;
{% endunless %}
{% endfor %}
</code></pre>
{% endraw %}

相关的 Jekyll issue:   
- [Why Jekyll convert my Capital words into lowercase in Categories](http://stackoverflow.com/questions/19074064/why-jekyll-convert-my-capital-words-into-lowercase-in-categories)   
- [jekyll/jekyll#842](https://github.com/jekyll/jekyll/issues/842)


## 8. warning: cannot close fd before spawn; 'which' is not recognized as an internal or external command, operable program or batch file

Windows环境下使用`pygments.rb`高亮code，即使plugin正常运行，但目前还存在这个麻烦的问题。因为是warning，所以本地运行Jekyll，一般可以忽略。   
然而博主由于强迫症猝发，本着追求完美的心态，就仔细追查了这个问题的根源。   
其实还是由Windows和Linux的系统环境造成的，确实比较麻烦。虽说解决方法有几个，目前 pygments.rb 0.6.0 仍然未得到解决。

![cannot close fd before spawn]({{ site.picture_dir }}/github-pages-issue/cannot_close_fd.png '"cannot close fd before spawn"')

相关的 GitHub Issue:   
- [tmm1/pygments.rb#90](https://github.com/tmm1/pygments.rb/pull/90)   
- [tmm1/pygments.rb#138](https://github.com/tmm1/pygments.rb/pull/138)   
- [rtomayko/posix-spawn#61](https://github.com/rtomayko/posix-spawn/issues/61)   
- [jekyll/jekyll#2789](https://github.com/jekyll/jekyll/issues/2789)   
- [jekyll/jekyll#2052](https://github.com/jekyll/jekyll/issues/2052)


## 9. 关于代码高亮

- 用js插件：[DlHightLight][1]或**[Google Code Prettify][2]**或<u>**[Highlight.js][3]**</u>或**[dp.SyntaxHighlighter][4]**
- 用gist：推荐菜鸟使用，省心省事，支持语言多
- 用pygment：要安装python以及python的包管理软件，定制code style CSS文件，又是个大坑，不建议菜鸟使用，尤其是使用windows的

[1]: http://mihai.bazon.net/projects/javascript-syntax-highlighting-engine
[2]: https://code.google.com/p/google-code-prettify/
[3]: https://github.com/isagalaev/highlight.js
[4]: http://alexgorbatchev.com/SyntaxHighlighter/


## 10. markdown extension

There are two ways to strengthen the code style:   
one is to indent 4 spaces or 1 tab;   
the other is to use {% raw %}`{% highlight %}`{% endraw %} tag to parse the code block.

Now GitHub provides another approach to do it: use triple backticks(`` ` ` ` ``) to format text as its own distinct block.

Yes, it's OK to use like that and the result is similar to highlight block. But I found that by the second way only `redcarpet` and `kramdown` can identify the code language and render it with html; `rdiscount` won't add specific html and css to improve the code style.
