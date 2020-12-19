---
layout: post
title: "Bash: Highlight Matched Texts With Colors"
---

Logs are valuable sources to identify issues. It would be continuously generated (e.g. `adb logcat`) or stored as a file. Mostly, I use the terminal to check those. When I check that, it is often not easy to see since every text has the same color. `grep` could help to filter out. With `--color=auto` or `--color=always` option, it supports highlighting texts as well. However, there are cases I don't want to filter out but want to highlight certain texts or words to check easily. This happens more frequently when I check continuous logs. 

After some investigation, I've figured out how to do it in bash. By adding the below function, you can highlight a matched text with a specified color.

<p class="code-label">.bash_profile</p>
{% highlight shell %}
highlight() {
  # Usage: highlight <color> <text>
  #
  # Foreground colors
  # ---
  # 30:black, 31:red, 32:green, 33:yellow, 34:blue, 35:magenta, 36:cyan

  # Background colors
  # ---
  # 40:black, 41:red, 42:green, 43:yellow, 44:blue, 45:magenta, 46:cyan
  escape=$(printf '\033')
  sed "s,$2,${escape}[$1m&${escape}[0m,g"
}
{% endhighlight %}

The actual showing color is determined by terminal settings. For example, _iTerm_ defines colors in _Profiles > Colors_.
![iTerm Colors](/images/2020/12-24/iterms-colors.png)

{% include google-adsense-in-article.html %}

Here is an example on how to use `highlight` and how the result looks.
{% highlight shell %}
$ echo Hello | highlight 31 H | highlight 32 e | highlight 33 l | highlight 34 o
{% endhighlight %}
![Highlight Hello](/images/2020/12-24/highlight-hello.png)


Though this function works well, it requires quite a long command if I want to highlight multiple texts, as you can see above. Specifying a color is great, but it becomes somewhat cumbersome if it is needed every time. All I need is just to highlight texts with different colors. Based on the `highlight` function, the below helper function simplifies it.
<p class="code-label">.bash_profile</p>
{% highlight shell %}
ht() {
  if [[ $# == 1 ]]; then
    highlight 31 $1
  fi
  if [[ $# == 2 ]]; then
    highlight 31 $1 | highlight 32 $2
  fi
  if [[ $# == 3 ]]; then
    highlight 31 $1 | highlight 32 $2 | highlight 33 $3
  fi
  if [[ $# == 4 ]]; then
    highlight 31 $1 | highlight 32 $2 | highlight 33 $3 | highlight 34 $4
  fi
}
{% endhighlight %}
![ht Logcat](/images/2020/12-24/ht-hello.png)
![ht Logcat](/images/2020/12-24/ht-logcat.png)

