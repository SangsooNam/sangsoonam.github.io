---
layout: post
title: "Using Git Worktree to Deploy GitHub Pages"
---

GitHub Pages is a website hosted directly from your GitHub repository. Generally, it is used together with static site generators. Static site generators (e.g. Jekyll, Hugo, and Gitbook) make the process of making a site easily and the generated HTML pages can be hosted on GitHub Pages.

When you use static site generators, there would be sources and generated files. For hosting on GitHub Pages, it needs generated files but not sources. GitHub Pages supports to select a source branch: `master` or `gh-pages`.  So, general configuration is to have sources in `master` branch and put generated files in `gh-pages` branch.

In this configuration, there is a cumbersome deploy work. Static site generators usually generate files in the subdirectory such as `_site`. Since there are two branches, `master` and `gh-pages`, you need to move them from `master` branch to the root directory of  `gh-pages` branch. It is required whenever you want to deploy.

Since Git 2.5, Git supports to manage multiple working trees attached to the same repository. With that, you can mount a git branch as a subdirectory. This makes a deploy process easier and it doesn't require `git checkout`. Let's see how to do it step by step.

{% include google-adsense-in-article.html %}

## Setup

First of all, you need to have a `gh-pages` branch which is going to be mounted. If you don't have, you can create a branch with `git branch`.

{% highlight bash %}
$ git branch gh-pages
{% endhighlight %}

This makes a branch based on the master HEAD. It would be okay but the files and the git history of `master` branch are not meaningful on `gh-pages` branch. Using an orphan branch, you can initialize `gh-pages` in a clean way.

{% highlight bash %}
$ git checkout --orphan gh-pages
$ git reset --hard
$ git commit --allow-empty -m "Init"
$ git checkout master
{% endhighlight %}

Then, you can mount the branch as a subdirectory using `git worktree`. Make sure that you don't have an existing target subdirectory.

{% highlight bash %}
$ rm -rf _site
$ git worktree add _site gh-pages
Preparing _site (identifier _site)
HEAD is now at b475e3e Initializing gh-pages branch
{% endhighlight %}

> If you didn't ignore `_site` in git, ignore it so that you don't add generated files accidentally.
{% highlight bash %}
$ echo "_site" >> .gitignore
{% endhighlight %}

When everything is set correctly, `git branch` shows a different branch for a root directory and a subdirectory.

{% highlight bash %}
$ git branch
  gh-pages
* master

$ cd _site
$ git branch
* gh-pages
  master
{% endhighlight %}

## Deploy

When you build a static site, generated files are in  `_site` directory. Since `_site` is now `gh-pages` branch, you can deploy by just creating a commit and pushing it.

{% highlight bash %}
$ cd _site
$ git add --all
$ git commit -m "Deploy updates"
$ git push origin gh-pages
{% endhighlight %}

## Clean Up

To unmount the subdirectory, you can run a below command. When you run, it will remove the worktree information and the mounted subdirectory.

{% highlight bash %}
$ git worktree remove _site
{% endhighlight %}

> If you delete the `_site` directory without `git worktree remove`, `git worktree` doesn't know it and keep showing it in the `git worktree list`. It will be eventually removed (`gc.worktreePruneExpire`). If you clean up immediately, run `git worktree prune`.

## Automate with Script

You now understand how to do it step by step. Now, it's time to automate with a shell script. Using below script, you just need to run it when you want to deploy.

{% gist SangsooNam/06de16f154d0e9a08b436a7252e80f34 %}
