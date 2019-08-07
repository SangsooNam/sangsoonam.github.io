---
layout: post
title: "Why ZSH is Slow for Git Repos"
---

In these days, I'm using ZSH([https://ohmyz.sh/](https://ohmyz.sh/)). It has many handy features and supports useful plugins. One day, I've noticed a slowness issue. Basically, the prompt was not shown right after running a command. It was interesting because it only happened in the git repositories. To fix this issue, I needed to understand what's happening first.

When you run `zsh` with a `--help` parameter, you can see many options. Among them, `-v` and `-x` are useful ones to dig into the problem.

{% highlight shell %}
$ zsh --help
...
-v    equivalent to --verbose
-x    equivalent to --xtrace
...
{% endhighlight %}

When you run `zsh -xv`, every command that ZSH runs will be shown. In the git repo, I pressed `enter` multiple times and checked logs to figure out which command is slow. There might be some better ways but I just looked at the terminal. I saw that there was a bit freeze when the below line was shown.

{% highlight shell %}
$ zsh -vx
...
+git_prompt_status:1> local INDEX STATUS
+git_prompt_status:2> INDEX=+git_prompt_status:2> git status --porcelain -b
...
{% endhighlight %}

{% include google-adsense-in-article.html %}

To show helpful information on the prompt, ZSH has many defined functions. Git related functions are defined in `git.zsh`. As you can see, that command is from `git_prompt_status()`.

<p class="code-label">.oh-my-zsh/lib/git.zsh</p>
{% highlight shell %}
...
# Get the status of the working tree
function git_prompt_status() {
  local INDEX STATUS
  INDEX=$(command git status --porcelain -b 2> /dev/null)
  STATUS=""
...
{% endhighlight %}

Those functions are generally called by ZSH themes and plugins. In my case, `avit` theme, I'm currently using, uses it to set `RPROMPT`.

<p class="code-label">.oh-my-zsh/themes/avit.zsh-theme</p>
{% highlight shell %}
...
RPROMPT='$(_vi_status){$(echotc UP 1)%}$(_git_time_since_commit) $(git_prompt_status) ${_return_status}%{$(echotc DO 1)%}'
...
{% endhighlight %}

It is possible to fix or optimize `git_prompt_status()` function in `git.zsh`. However, I realized that it was the information that I didn't need actually. So, I decided to remove that. One way is to remove that line in the theme directory. `.oh_my_zsh` is a git repo. So, modifying the code in there could make an issue to update ZSH. Instead of that, I simply override `RPROMPT` value in `.zshrc`. This is a safer approach.

<p class="code-label">.zshrc</p>
{% highlight shell %}
...
RPROMPT=''
...
{% endhighlight %}

Note that a problem line could vary depending on your theme and plugins. When you have some issues, `zsh -vx` will help you to point a problem command.
