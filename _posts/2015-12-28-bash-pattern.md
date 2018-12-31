---
layout: post
title: "Bash Pattern"
---

## Variable

A variable is a symbolic name for information. In the bash, it can be declared with an equal sign `=`. Unlike other languages such as C and Java, a variable type is not needed.

{% highlight sh %}
❯ VAR=28
{% endhighlight %}

To use/expand the variable, you can simply type its name used in declaration with a prefix `$` . If you use the variable with joining a value, you need to wrap the variable explicitly with `${}`

{% highlight sh %}
❯ echo $VAR
28
❯ echo ${VAR}iable
28iable
{% endhighlight %}

Variables could reference any value. There is one thing you have to care. When you declare a variable to reference a string, the string could have white spaces. To store the string and reference it properly, it is necessary to wrap with double quotes `""`. If a string doesn't have any white spaces, it is an optional.

{% highlight sh %}
❯ VAR="A B C"
❯ echo $VAR
A B C
{% endhighlight %}


{% include google-adsense-in-article.html %}

## Pattern Removal

Once you declare a variable, there are several operations to modify a expanded value. A pattern removal operation is one of them. In here, a pattern is different from a regular expression. The pattern has special characters having the following meanings:

* \* Matches any string, including the null string.
* ? Matches any single character.
* [...] Matches any one of the enclosed characters.

`%` is a pattern removal operator to remove a trailing string. If there is no matched string, it will return the original string.

{% highlight sh %}
❯ REPLY="HEY! HELLO"
❯ echo ${REPLY%O}
HEY! HELL
❯ echo ${REPLY%?}
HEY! HELL
❯ echo ${REPLY%[oO]}
HEY! HELL
❯ echo ${REPLY%X}
HEY! HELLO
{% endhighlight %}

There are two pattern removal operators. One is `%` and the other is `%%`. `%` supports a shortest pattern matching and `%%` supports a longest pattern matching. Here are examples.

{% highlight sh %}
❯ echo ${REPLY%L*}
HEY! HEL
❯ echo ${REPLY%%L*}
HEY! HE
{% endhighlight %}

`#` is a similar operator like `%`. The only difference is that it is to remove a beginning string. There are also similar two operators: `#` and `##`

{% highlight sh %}
❯ REPLY="HEY! HELLO"
❯ echo ${REPLY#H}
EY! HELLO
❯ echo ${REPLY#*H}
EY! HELLO
❯ echo ${REPLY##*H}
ELLO
{% endhighlight %}

With a pattern removal operation, it would be an easy task to replace an extension of a given filename.

{% highlight sh %}
❯ FILENAME="music.avi"
❯ echo ${FILENAME} ${FILENAME%.*}.mp3
music.avi music.mp3
❯ mv ${FILENAME} ${FILENAME%.*}.mp3
{% endhighlight %}

## Pattern Replacement

If you want to replace a string, you can utilize a pattern replacement operation using `${variable/pattern/string}`. If a expanded string of variable contains a matched pattern, a matched part will be replaced with a input string.

{% highlight sh %}
❯ FILENAME="music.avi"
❯ echo ${FILENAME} ${FILENAME/.avi/.mp3}
music.avi music.mp3
{% endhighlight %}

By default, only the first match is replaced. To replace every match, the pattern should begin with `/`.

{% highlight sh %}
❯ REPLY="HEY! HELLO"
❯ echo ${REPLY/HE/__}
__Y! HELLO
❯ echo ${REPLY//HE/__}
__Y! __LLO
{% endhighlight %}

If pattern begins with `%`, it must match at the end of the expanded string of variable.

{% highlight sh %}
❯ REPEAT="HELLO HELLO"
❯ echo ${REPEAT/HELLO/hello}
hello HELLO
❯ echo ${REPEAT/%HELLO/hello}
HELLO hello
{% endhighlight %}

If pattern begins with `#`, it must match at the beginning of the expanded string of variable. As you know, the original string will be returned if there is no matched pattern. In this example, `REPEAT` value doesn't begin with `HELLO` so it returns the original string for the `#HELLO` pattern.

{% highlight sh %}
❯ REPEAT="HEY! HELLO HELLO"
❯ echo ${REPEAT/HELLO/hello}
HEY! hello HELLO
❯ echo ${REPEAT/#HELLO/hello}
HEY! HELLO HELLO
{% endhighlight %}

> To get more information, type `man bash` and see the 'Parameter Expansion' and 'Pattern Matching' sections.
