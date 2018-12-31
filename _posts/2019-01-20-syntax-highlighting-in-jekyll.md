---
layout: post
title: "Syntax Highlighting in Jekyll"
---

Jekyll has powerful support for syntax highlighting to show code snippets better. By default, Jekyll utilizes [Rouge](http://rouge.jneen.net/), which is a pure ruby-based code highlighter. Rouge can highlight 100 different languages. Thus, you don't have to worry about how to set up something for syntax highlighting.

![Rouge](/images/2019/01-15/rouge.png)

## How to Use
When you want to show the code snippet, you can enable syntax highlighting with `{% raw %}{% highlight <lang> [linenos]%}{% endraw %}` and `{% raw %}{% endhighlight %}{% endraw %}`. `lang` is a required parameter. You can check [a list of supported languages](https://github.com/jneen/rouge/wiki/List-of-supported-languages-and-lexers) and [live demo](http://rouge.jneen.net/).

{% highlight plaintext %}
{% raw %}
{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}
{% endraw %}
{% endhighlight %}


{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

With an optional `linenos` parameter, the line numbers are shown while highlighting the code.

{% highlight plaintext %}
{% raw %}
{% highlight ruby linenos %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}
{% endraw %}
{% endhighlight %}

{% highlight ruby linenos %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

{% include google-adsense-in-article.html %}

## fenced_code_blocks for Markdown

You typically write posts in Markdown, supported by Jekyll. In Markdown, syntax highlighting is enabled with three backticks(<code class="highlighter-rouge">&#96;&#96;&#96;</code>). It is natural that you want to do it instead of `{% raw %}{% highlight <lang> [linenos]%}{% endraw %}`. This is called `fenced_code_blocks` and Jekyll set it to `true` by default. So, you can do like below:


{% highlight plaintext %}
{% raw %}
```ruby
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
```
{% endraw %}
{% endhighlight %}

You may assume that this is a just simpler way for syntax highlighting in Jekyll. It looks like that but there are a few small differences actually.

* **Line numbers**: With `fenced_code_blocks`, you can't specify `linenos` to show the line numbers together.
* **HTML Output**: There are different HTML outputs. Although both are compatible with themes, you might need to consider this once you add some custom styling.

_With highlight and endhighlight_
![highlight html](/images/2019/01-15/highlight.png)

_With fenced_code_blocks_
![fenced_code_blocks html](/images/2019/01-15/fenced_code_blocks.png)


## Themes

Rouge's HTML output is compatible with stylesheets designed for [Pygmenets](http://pygments.org/), which is another syntax highlighter written in Python. It was released much earlier than Rouge. Rouge's compatibility makes us find available stylesheets easily on the internet. You can find some in [Jekyll Themes](http://jwarby.github.io/jekyll-pygments-themes/languages/java.html). After downloading a stylesheet, copy it to a CSS directory (`/css`). Then, you can import it in the main stylesheet like this:

{% highlight css %}
@import "fruity.css"
{% endhighlight %}
