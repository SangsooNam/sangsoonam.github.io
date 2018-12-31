---
layout: post
title: "Create a Pull Request using Command Line Tool, Hub"
---

GitHub is a decent version control repository for collaboration. To work together, GitHub suggests a flow, including to make a pull request. Here is the definition of pull request from ["About pull requests"](https://help.github.com/articles/about-pull-requests/) GitHub.

> Pull requests let you tell others about changes you've pushed to a repository on GitHub. Once a pull request is opened, you can discuss and review the potential changes with collaborators and add follow-up commits before the changes are merged into the repository.

General way to make a pull request is that you first change the code to suggest in your branch, push your branch, and click a create pull request button in the GitHub.

_Image from ["Creating a pull request"](https://help.github.com/articles/creating-a-pull-request)_

![Create a pull request](/images/2016/02-01/pull-request-review-edit-branch.png)

This is not a complicate. However, you need to open the web browser to do it. For command line lovers, GitHub made a commland-line wrapper for `git`, which is called `hub`([https://hub.github.com](https://hub.github.com/)).

{% highlight sh %}
# OS : install the latest release
❯ brew install hub
# alias it as git
❯ alias git=hub
{% endhighlight %}

{% include google-adsense-in-article.html %}

After this, you can run GitHub commands in addition to normal `git` commands.

{% highlight sh %}
❯ git
...
These GitHub commands are provided by hub:

   pull-request   Open a pull request on GitHub
   fork           Make a fork of a remote repository on GitHub and add as remote
   create         Create this repository on GitHub and add GitHub as origin
   browse         Open a GitHub page in the default browser
   compare        Open a compare page on GitHub
   release        List or create releases
   issue          List or create issues
   ci-status      Show the CI status of a commit
{% endhighlight %}

Without opening a web browser, now we can make a pull request using a command line.

{% highlight sh %}
❯ git co -b feature_branch
# Modify the code
❯ git add .
❯ git commit
❯ git push
# Create a pull request using hub
❯ git pull-request
{% endhighlight %}

Everything works well. However, I think one thing is missing. When you run `git pull-request`, an editor, such as `vi`, will be opened, and you need to type a pull request message. When you use a web browser, that is the same message from the last commit if you made only one commit change. Since that is the most common case, I don't want to type again every time.

Fortunately, `hub` allows us to set a message parameter when run the command.

{% highlight sh %}
❯ git pull-request --help

Usage: hub pull-request [-foc] [-b <BASE>] [-h <HEAD>] [-a <USERS>] [-M <MILESTONE>] [-l <LABELS>]
       hub pull-request -m <MESSAGE>
       hub pull-request -F <FILE> [--edit]
       hub pull-request -i <ISSUE>
{% endhighlight %}

So, only left thing is to get the last commit message. We can get it using a below command.

{% highlight sh %}
❯ git log -1 --pretty=%B
{% endhighlight %}

To sum up, we can run below command to create a pull request which having a pull request message from the last commit without opening a web browser and an editor.

{% highlight sh %}
❯ git pull-request -m "$(git log -1 --pretty=%B)"
{% endhighlight %}

Of course, `alias` is always our best friends.
{% highlight sh %}
alias gpr='git pull-request -m "$(git log -1 --pretty=%B)"'
{% endhighlight %}
