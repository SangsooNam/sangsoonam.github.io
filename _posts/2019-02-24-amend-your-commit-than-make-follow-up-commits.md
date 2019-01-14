---
layout: post
title: "Amend Your Commit than Make Follow-up Commits"
---

GitHub supports good ways to collaborate with other engineers. Especially, pull requests let you tell others about your changes. Based on a pull request, you can get a review from others or can discuss. When reviewers give you feedback, you can create another commit to apply it. These commits are called follow-up commits.

This is a typical and [suggested flow](https://help.github.com/articles/about-pull-requests) in GitHub. Follow-up commits let reviewers check changes easily after their feedback. Since there are new commits, they don't need to check all files and can check just those commits. Moreover, follow-up commits don't change the git history. Although there are several benefits, I don't think this is a good practice. The problem is that follow-up commits could be like this:

{% highlight bash %}
a9bc78a (HEAD -> auto-login) Add a missing annotation
bd29011 Rename a method
c4881fb Fix a typo
e10bd31 Add a auto login feature
{% endhighlight %}

{% include google-adsense-in-article.html %}

**It is really important for you to know that the git history consists of git commits, not pull requests.** A pull request is just a handy way to discuss and get feedback supported by GitHub. Each commit doesn't have anything on the pull request.

Once a pull request is merged, follow-up commits will be a part of git history. This makes the git history longer and verbose. Furthermore, it makes hard to find a problem point when you have an issue. Since there is no remaining context on a pull request, it is often hard to know why this commit was made.

To avoid this, I suggest to amend your commit rather than adding follow-up commits. You can amend a commit with `--amend`.

{% highlight bash %}
$ git add .
$ git commit --amend
{% endhighlight %}

Technically, `--amend` doesn't change the content of a previous commit. It makes a new commit and replaces the previous commit in the git history. When you run `git log`, you can see the new commit having a different git hash after amending.

{% highlight bash %}
$ git add .
$ git log -1 --oneline
a9bc78a (HEAD -> auto-login) Add a auto login feature
$ git commit --amend
$ git log -1 --oneline
7c5fd69 (HEAD -> auto-login) Add a auto login feature
{% endhighlight %}

Since amending changes the git history, `git push` will fail. It doesn't allow to override the history by default. You can override the history in the remote with `--force` when you run `git push`.

{% highlight bash %}
$ git push
To github.com:SangsooNam/android-application.git
 ! [rejected]        master -> master (non-fast-forward)
{% endhighlight %}

{% highlight bash %}
$ git push --force
{% endhighlight %}

Then, the git history would be shorter and more concise. Nowadays, GitHub gives a way to keep a simple git history along with follow-up commits. When you merge, you can select `Squash and merge` instead of `Create a merge commit`.

![squash and merge](/images/2019/02-24/squash-and-merge.png)

It could make the same result but I consider amending a git commit is better because of:

* **No squashing**: It is possible to merge without squashing. You need to change it before selecting. Reviewers also don't know whether you will squash and merge.

* **Wrong commit message**: When you select `Squash and merge`, GitHub asks a new commit message. By default, it contains a title and a message of each commit. Since titles and messages from follow-up commits are not useful, you need to clean up the message manually. If not, a squashed commit could have a wrong or meaningless message.

![squash commit messages](/images/2019/02-24/squash-commit-messages.png)


## Conclusion

The git history is based on git commits. When you get feedback in your pull request, you can make changes with follow-up commits. That makes the git history lengthy and hard to read. I suggest amending your commit instead of making follow-up commits. You can do it with `git commit --amend`.
