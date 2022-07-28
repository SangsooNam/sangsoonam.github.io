---
layout: post
title: "Git: Restore a Commit with Reflog"
---

`git reset` is generally used to cancel a commit. It happens mainly for two reasons. One is just to cancel it. The other is to modify changes. After changes, a new commit can be created. `git commit --amend` is an alternative way to modify it. Both ways can make the same result. 

{% highlight shell %}
▶ git log --oneline
a91929a (HEAD -> master) Add skeleton files
6e9be90 Add .gitignore
be44318 Init

▶ git reset HEAD^

▶ git log --oneline
6e9be90 (HEAD -> master) Add .gitignore
be44318 Init

▶ git st
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
  package.json
  yarn.lock
  app.tsx
  index.html

nothing added to commit but untracked files present (use "git add" to track)

{% endhighlight %}

These commands are quite useful and powerful. The thing is that I sometimes want to cancel those and restore the previous commit.  After `git reset`, it is gone. `git log` doesn't show it anymore.

{% highlight shell %}
▶ git log --oneline
6e9be90 (HEAD -> master) Add .gitignore
be44318 Init
{% endhighlight %}

{% include google-adsense-in-article.html %}

Then, what would be the way to restore it? Maybe you pushed a branch or a commit to a remote repository before. When `git reset` happens only in a local machine, it's possible to get or restore it from the remote repository. If not, it can be tricky.. Luckily, `git` has reference logs. The `git` document describes 

> Reference logs, or "reflogs", record when the tips of branches and other references were updated in the local repository. 

Basically, `git` doesn't remove commits right away when `git reset` happen. It changes references first. In this example, `HEAD` is now changed to `6e9be90`. `git reflog` shows those reference change logs. These logs are stored only locally and `git` will prune reference logs older than 90 days by default. The below command shows the commit id, `a91929a`, before running `git reset HEAD^`.


{% highlight shell %}
▶ git reflog
6e9be90 (HEAD -> master) HEAD@{0}: reset: moving to HEAD^
a91929a HEAD@{1}: commit: Add skeleton files
6e9be90 (HEAD -> master) HEAD@{2}: commit: Add .gitignore
be44318 HEAD@{3}: commit (initial): Init
{% endhighlight %}

`git reset --hard <COMMIT_ID>` would restore that commit and references.

{% highlight shell %}
▶ git reset --hard a91929a
HEAD is now at a91929a Add skeleton files

▶ git log --oneline
a91929a (HEAD -> master) Add skeleton files
6e9be90 Add .gitignore
be44318 Init
{% endhighlight %}

The same way is applicable to restore after `git commit --amend`. This command is used to apply more changes to the existing commit. Technically, `git commit --amend` does't modify an existing commit. It will create a new commit using the changes of the most recent existing commit. Then, `git` adjust references to show only the new commit. `git log` doesn't show that previously existing commit.


{% highlight shell %}
▶ git commit --amend
[master 447608a] Add skeleton files
 ...

▶ git log --oneline
447608a (HEAD -> master) Add skeleton files
6e9be90 Add .gitignore
be44318 Init
{% endhighlight %}

It means that it would be easy to restore by knowing that previously existing commit id. `git reflog` shows those records. The commit id before amending is `a91929a`. 

{% highlight shell %}
▶ git reflog
447608a (HEAD -> master) HEAD@{0}: commit (amend): Add skeleton files
a91929a HEAD@{1}: reset: moving to a91929a
6e9be90 HEAD@{2}: reset: moving to HEAD^
a91929a HEAD@{3}: commit: Add skeleton files
6e9be90 HEAD@{4}: commit: Add .gitignore
be44318 HEAD@{5}: commit (initial): Init
{% endhighlight %}

`git reset --hard` command will restore that commit and references. 

{% highlight shell %}
▶ git reset --hard a91929a
HEAD is now at a91929a Add skeleton files

▶ git log --oneline
a91929a (HEAD -> master) Add skeleton files
6e9be90 Add .gitignore
be44318 Init
{% endhighlight %}

One more thing. If you want to learn more on references, try `git cat-file -p <COMMIT_ID|BRANCH>`.
{% highlight shell %}
# After `git reset --hard a91929a`, the references are the same as `a91929a`
▶ git cat-file -p master
tree 3f5d073729ffc62a7a1f9e307ba510a3f4762ea9
parent 6e9be90d94f41e176f1a5a9a706e14fdb10f1586

# `a91929a` have a parent, `6e9be90`
▶ git cat-file -p a91929a
tree 3f5d073729ffc62a7a1f9e307ba510a3f4762ea9
parent 6e9be90d94f41e176f1a5a9a706e14fdb10f1586

# Previously amend commit has the same parent, `6e89e90`
▶ git cat-file -p 447608a
tree 3f5d073729ffc62a7a1f9e307ba510a3f4762ea9
parent 6e9be90d94f41e176f1a5a9a706e14fdb10f1586
{% endhighlight %}