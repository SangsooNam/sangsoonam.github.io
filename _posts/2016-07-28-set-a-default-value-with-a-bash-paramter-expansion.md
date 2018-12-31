---
layout: post
title: "Set a Default Value with a Bash Parameter Expansion"
---

Bash scripts are commonly used to make repetitive tasks easier. Let's say you do this.

{% highlight sh %}
❯ tar -cf backup.tar /data/logs
{% endhighlight %}

Instead of typing this every time, you can create a bash script, called `backup.sh`

{% highlight sh %}
#!/bin/bash
tar -cf backup.tar /data/logs
{% endhighlight %}

Then, you just need to type the bash script name to execute it.

{% highlight sh %}
❯ ./backup.sh
{% endhighlight %}

It seems just a bit shorter, but you don't need worry about any typo. It would become much more powerful when you typed a long command previously.

{% include google-adsense-in-article.html %}

The backup file is generated in the current folder. If you want to change it dynamically, how can you do it? The easiest way to do it is using argument variables. Those are set when you execute the script. Here you can see one argument variable, `/my-backup/`.

{% highlight sh %}
❯ ./backup.sh /my-backup/
{% endhighlight %}

You can set many argument variables which are separated by a whitespace. Then, those argument variables can be accessed in the script. For accessing the first argument variable, type `$1` or `${1}`. For accessing the second argument variable, type `$2` or `${2}`. Here is the modified script with the argument variable.

{% highlight sh %}
#!/bin/bash
tar -cf ${1}backup.tar /data/logs
{% endhighlight %}

Now, the backup file will be generated in the `/my-backup/` directory when you set as an argument variable. If you don't set the argument variable, it is regarded as an empty string so that the backup file will be generated in the current directory.

Most of the cases, these are enough. However, you may often want to set the default directory where the backup will be generated. If you are a programmer, you probably think something like a `if` statement. Bash handles it in a much simpler way. That is called a bash parameter expansion.

{% highlight sh %}
${PARAMETER:-DEFAULTVALUE}
{% endhighlight %}

If the parameter has a value, it will be used. Otherwise, the default value will be used. To set the `/backup/` as a default directory, you can modify the script like below.

{% highlight sh %}
#!/bin/bash
tar -cf ${1:-/backup/}backup.tar /data/logs
{% endhighlight %}
