---
layout: post
title: "Checkout Pull Requests from GitHub"
---

GitHub give us an effective way to collaborate on development projects. Developers easily can participate on a project and contribute. The way to contribute is often called as the pull request workflow. When developers want to suggest some code change, they modify the code and can send code change which is called a pull request. Project maintainers, including you, can review the set of changes, discuss and merge those.

![Pull Requests](/images/2016/06-18/pull-requests.png)

Usual way to review is by looking. However, sometimes you may want to run the modified code instead of just looking. How can you do it? Unfortunately, you cannot simply checkout and run it because it was not made by you. In order to checkout it, you need to specify a remote repository name and a branch name. Because there are many contributers, `<user_name>` could be different for each pull request and could be a quite tedious task.

{% highlight sh %}
$ git pull git@github.com:<user_name>/<repository.git> <branch_name>
{% endhighlight %}

What is the better way? In fact, GitHub creates a special branch for each pull request. You can refer it as `refs/pull/<pull_request_number>/head`. Thus, you can easily checkout it when you know the pull request number.

{% highlight sh %}
$ git pull upstream pull/2637/head:pr/2637
From github.com
 * [new ref]         refs/pull/2637/head -> pr/2637
$ git checkout pr/2637
{% endhighlight %}

{% include google-adsense-in-article.html %}

If you do this frequently, you probably will consider git aliases. Someone([https://gist.github.com/gnarf/5406589](https://gist.github.com/gnarf/5406589){:target="_blank"}) already made those aliases, and you can just use it.


{% highlight sh %}
[alias]
  pr  = "!f() { git fetch -fu ${2:-upstream} refs/pull/$1/head:pr/$1 && git checkout pr/$1; }; f"
  pr-clean = "!git for-each-ref refs/heads/pr/* --format='%(refname)' | while read ref ; do branch=${ref#refs/heads/} ; git branch -D $branch ; done"
{% endhighlight %}

There are two aliases. `pr` is to fetch the pull request branch and checkout. `pr-clean` is to delete every fetched pull request branch.

{% highlight sh %}
❯ git pr 2637
remote: Counting objects: 21, done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 21 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (21/21), done.
From https://github.com/square/okhttp
 * [new ref]         refs/pull/2637/head -> pr/2637
Switched to branch 'pr/2637'
{% endhighlight %}

{% highlight sh %}
❯ git pr-clean
Deleted branch pr/2637 (was f06cf04).
{% endhighlight %}
