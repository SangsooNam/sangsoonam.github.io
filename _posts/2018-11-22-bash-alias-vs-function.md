---
layout: post
title: "Bash: Alias vs Function"
---

When you use a command line interface a lot, there would be some commands you use every day. Some commands are short but the others could be a bit longer. One reason is that one command runs with several parameters. Another reason is that one command can consist of multiple commands with some operators (e.g. `|`, `:`, `&&`). In any case, that is cumbersome to type the same command, especially for longer ones.

Bash shell has two ways for this: _alias_ and _function_. Alias is a simple way to create a shortcut. The format for creating it and some examples follow:

<p class="code-label">.bash_profile</p>
{% highlight bash %}
# alias name='command'
alias gbr='git branch --remote'
alias gco='git checkout'
alias l='ls -lah'
{% endhighlight %}

To run the alias, type the alias name in your bash shell.

{% highlight bash %}
$ gbr
  origin/HEAD -> origin/master
  origin/master
  origin/release
{% endhighlight %}

Most aliases contain every information to run the command. Thus, the name would be the only thing you need to type. In some cases, however, you want to run it together with parameters. `gco` is the one example. You want to shorten the typing instead of `git checkout` and you also want to specify the branch when you run it. For that, you just need to type parameters after the alias. Commands below works exactly the same.

{% highlight bash %}
$ gco release
$ git checkout release
{% endhighlight %}

Alias acts as replacing a name to a defined value. Although it is quite handy for most cases, you might want to do something more advanced where alias doesn't fit well.

{% include google-adsense-in-article.html %}

First, you cannot specify the parameter position with alias. Alias always put parameters at the end of the defined command. Below commands has the same command and option except for `MainActivityTest` and `PlayerActivityTest`. If you want to replace that part, alias doesn't support it

{% highlight bash %}
$ ./gradlew :app:testDebugUnitTest --tests="*.MainActivityTest"
$ ./gradlew :app:testDebugUnitTest --tests="*.PlayerActivityTest"
{% endhighlight %}

That is the time you need to consider functions. Function allows specifying the parameter position. Parameters are referred with numbers followed by `$` character.

<p class="code-label">.bash_profile</p>
{% highlight bash %}
# function_name() {
#   body
# }
ut() {
  ./gradlew :app:testDebugUnitTest --tests="*.$1"
}
{% endhighlight %}

To run the function, you type the function name. By seeing the command, it is hard to know whether it runs an alias or a function.

{% highlight bash %}
$ ut MainActivityTest
$ ut PlayerActivityTest
{% endhighlight %}

Second, alias requires to escape special characters. Let's see the below example. With `awk` command, that command shows the male employee names only.

{% highlight bash %}
$ cat data
  m broderick
  f amelia
  f julie
  m samual

$ cat data | awk '$1 =="m"{print $2;}'
  broderick
  samual
{% endhighlight %}

If you make an alias, you need to wrap with a single(or double) quote the command. It would be like this:

<p class="code-label">.bash_profile</p>
{% highlight bash %}
alias male='cat data | awk '$1 =="m"{print $2;}''
{% endhighlight %}

When the alias is loaded, there is an error. The reason is that there are single quotes in the value.

{% highlight bash %}
=m{print not found
{% endhighlight %}

To escape it properly, you need to put `'\''` instead of `\'` because of the bash interpretation.

<p class="code-label">.bash_profile</p>
{% highlight bash %}
alias male='cat data | awk '\''$1 =="m"{print $2;}'\'''
{% endhighlight %}

Although it is doable, the original command gets hard to read and maintain. Function fits better in this case. Just make a function and put the command in the function body. You don't need to worry about escaping anymore!

<p class="code-label">.bash_profile</p>
{% highlight bash %}
male() {
  cat data | awk '$1 =="m"{print $2;}'
}
{% endhighlight %}
