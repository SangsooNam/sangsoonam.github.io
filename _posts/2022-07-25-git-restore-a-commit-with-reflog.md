---
layout: post
title: "Git: Restore a Commit with Reflog"
---

`git reset` is generally used to cancel a commit. It happens mainly for two reasons. One is just to cancel it. The other is to modify changes. After changes, a new commit can be created. `git commit --amend` is an alternative way to modify it. Both ways can make the same result. 

{% highlight shell %}
▶ git log --oneline
b55d6c4 (HEAD -> master) Add files
65550b9 Add a config file
0da17c6 init

▶ git reset HEAD^

▶ git log --oneline
65550b9 (HEAD -> master) Add a config file
0da17c6 init

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
b55d6c4 (HEAD -> master) Add files
65550b9 Add a config file
0da17c6 init

▶ git reset HEAD^

▶ git log --oneline
65550b9 (HEAD -> master) Add a config file
0da17c6 init

{% endhighlight %}

{% include google-adsense-in-article.html %}

Then, what would be the way to restore it? Maybe you pushed a branch or a commit to a remote repository before. When `git reset` happens only in a local machine, it's possible to get or restore it from the remote repository. If not, it can be tricky.. Luckily, `git` has reference logs. The `git` document describes 

> Reference logs, or "reflogs", record when the tips of branches and other references were updated in the local repository. 

Basically, `git` doesn't remove commits right away when `git reset` happen. It changes references first. In this example, `HEAD` is now changed to `65550b9`. `git reflog` shows those reference change logs. These logs are stored only locally and `git` will prune reference logs older than 90 days by default. The below command shows the commit id, `b55d6c4`, before running `git reset HEAD^`.


{% highlight shell %}
▶ git reflog
65550b9 (HEAD -> master) HEAD@{0}: reset: moving to HEAD^
b55d6c4 HEAD@{1}: commit: Add files
65550b9 (HEAD -> master) HEAD@{2}: commit: Add a config file
0da17c6 HEAD@{3}: commit (initial): init

{% endhighlight %}

`git reset --hard <COMMIT_ID>` would restore that commit and references.

{% highlight shell %}
▶ git reset --hard b55d6c4
HEAD is now at b55d6c4 Add files

▶ git log --oneline
b55d6c4 (HEAD -> master) Add files
65550b9 Add a config file
0da17c6 init
{% endhighlight %}


The same way is applicable to restore after `git commit --amend`. This command is used to apply more changes to the existing commit. Technically, `git commit --amend` does't modify an existing commit. It will create a new commit using the changes of the most recent existing commit. Then, `git` adjust references to show only the new commit. `git log` doesn't show that previously existing commit.


{% highlight shell %}
▶ git commit --amend
[master bb92f27] Add files
 ...

▶ git log --oneline
bb92f27 (HEAD -> master) Add files
65550b9 Add a config file
0da17c6 init
{% endhighlight %}

It means that it would be easy to restore by knowing that previously existing commit id. `git reflog` shows those records. The commit id before amending is `b55d6c4`. `git reset --hard` command will restore that commit and references. 

{% highlight shell %}
▶ git reset --hard b55d6c4
HEAD is now at b55d6c4 Add files

▶ git log --oneline
b55d6c4 (HEAD -> master) Add files
65550b9 Add a config file
0da17c6 init
{% endhighlight %}

One more thing. If you want to learn more on references, try `git cat-file -p <COMMIT_ID|BRANCH>`.
{% highlight shell %}
# After `git reset --hard b55d6c4`, the references are the same as `b55d6c4`
▶ git cat-file -p master
tree af6e6c5c77580cdec1c2fa9f112490b5ef06a40c
parent 65550b912a6edec8762843cd7b8aeb11df0fa82e

# Both have the same parent, `65550b9`
▶ git cat-file -p b55d6c4
tree af6e6c5c77580cdec1c2fa9f112490b5ef06a40c
parent 65550b912a6edec8762843cd7b8aeb11df0fa82e

▶ git cat-file -p bb92f27
tree 575e04f8838d1bcb380d05f2d42b5503db880a7b
parent 65550b912a6edec8762843cd7b8aeb11df0fa82e
{% endhighlight %}