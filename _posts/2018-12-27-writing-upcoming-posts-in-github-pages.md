---
layout: post
title: "Writing Upcoming Posts in GitHub Pages"
---

[GitHub Pages](https://pages.github.com) supports to make a website for you or your project. This is directly hosted by your GitHub repository. For blogging, it uses [Jekyll](https://jekyllrb.com/), which supports Markdown syntax and doesn't require any database. When you add and push a post to your repository, it will be published soon.

If you write several posts at a time, you probably want to publish them in the future instead of showing them at once. Jekyll supports below ways for that.

## Published
On each post, you can specify whether you want to publish it or not.

{% highlight yaml %}
---
layout: post
title: "title"
published: false
---
{% endhighlight %}

If you want to see unpublished posts together locally, use `--unpublished` option.

{% highlight bash %}
$ jekyll serve --unpublished
{% endhighlight %}

You can change the `_config.yml` to show unpublished posts without `--unpublished` option. However, I suggest not to change the config file to avoid showing unpublished posts to others.

<p class="code-label">_config.yml</p>
{% highlight yaml %}
unpublished: true
{% endhighlight %}

{% include google-adsense-in-article.html %}

## Future

In Jekyll, a post date is defined by a filename, e.g. `_posts/2018-08-21-apples.md`. If you use the future date on your post, it would be published on that date. To see future dated posts locally, you need to use `--future` option.

{% highlight bash %}
$ jekyll serve --future
{% endhighlight %}

You also can change the config file to show future posts always. I don't think you want to show those. In that case, it is vital to set it as `false`. When you run `jekyll serve` without `--future`, it won't show future posts. It is true when you run it locally. However, your future posts will be shown in GitHub Pages. I couldn't check the detail but it seemed the default Jekyll setting is different between a local and GitHub Pages. **So, please don't forget to set `false` to hide future posts correctly.**

<p class="code-label">_config.yml</p>
{% highlight yaml %}
future: false
{% endhighlight %}

> GitHub Pages doesn't update the content automatically. The content is updated only when there is a new commit. If you committed a post with a future date, it won't be shown on that date. You can trigger the update by pushing a new commit. If you don't have any new commit, you can amend the last commit and then push it for triggering.

{% highlight bash %}
$ git commit --amend --no-edit
$ git push --force
{% endhighlight %}

## Draft

If you put your post in `_drafts`, you are able to push it to GitHub along with hiding to others. When you move it into `_posts`, that will be shown. To see it locally, use `--drafts` option.

{% highlight bash %}
$ jekyll serve --drafts
{% endhighlight %}
