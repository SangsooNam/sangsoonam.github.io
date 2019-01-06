---
layout: post
title: "Random Related Posts in Jekyll"
---

Related posts are suggestions that could be interesting for readers who read the current post. This is quite useful since users can find more relevant content easily.

## Related Posts vs Recent Posts

In Jekyll, there is a `site.related_posts` variable.  Using it, related posts are rendered. This sounds simple. However, if you look closer the result, you can notice that it is not related posts, but recent posts.

{% highlight java %}
{% raw %}
{% for post in site.related_posts %}
  {% include related-post.html %}
{% endfor %}
{% endraw %}
{% endhighlight %}

This is because of the default Jekyll configuration. Here is a description from Jekyll:

> _`site.related_posts`_
>
> _If the page being processed is a Post, this contains a list of up to ten related Posts. By default, these are the ten most recent posts. For high quality but slow to compute results, run the jekyll command with the `--lsi` (latent semantic indexing) option._
>
> _[Jekyll site variables](https://jekyllrb.com/docs/variables/#site-variables)_


{% include google-adsense-in-article.html %}

## Related Posts with Latent Semantic Indexing

Latent semantic indexing is one of the most popular algorithms to calculate a similarity between documents. By enabling it, we can get the correct related posts. In Jekyll, you run with the `--lsi` or enable it in the `_config.xml`. When it is enabled, it shows `Populating LSI...`.

<p class="code-label">_config.xml</p>
{% highlight yaml %}
lsi: true
{% endhighlight %}

{% highlight bash %}
$ jekyll build
Configuration file: /Users/sangsoo/git/sangsoonam.github.io/_config.yml
            Source: /Users/sangsoo/git/sangsoonam.github.io
       Destination: /Users/sangsoo/git/sangsoonam.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 1.091 seconds.
 Auto-regeneration: disabled. Use --watch to enable.

$ jekyll build --lsi
Configuration file: /Users/sangsoo/git/sangsoonam.github.io/_config.yml
            Source: /Users/sangsoo/git/sangsoonam.github.io
       Destination: /Users/sangsoo/git/sangsoonam.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
  Populating LSI...
Rebuilding index...
                    done in 36.561 seconds.
 Auto-regeneration: disabled. Use --watch to enable.
{% endhighlight %}

Now, you can see the correct related posts. Although this brings a better result, this is quite slow. **In my case, it takes 36 times slower than normal build.** Moreover, this has a bigger problem if you host your blog on GitHub Pages.

## Problem on GitHub Pages

**Simply, GitHub Pages does not support the `lsi` option when generating sites.** There is no clear reason but I guess the performance is one of them. As you saw, it is much slower than normal build. GitHub Pages is currently free and LSI uses resource a lot.

GitHub Pages acts differently depending on source types: Jekyll and HTML. For Jekyll, it will generate a site and deploy it. For HTML, however, the source will be just deployed without a generation stage. Generally, `gh-pages` branch is used for the HTML source. It means that you can use the LSI if you push a locally generated site and change the source type. However, you need to push a generated site every time by yourself.


## Random Related Posts

There are some techniques to solve this GitHub Pages limitation by utilizing Jekyll tags or categories. [One](https://alligator.io/jekyll/related-posts-in-jekyll/) shows recent posts in the same category. [Another](https://blog.webjeda.com/jekyll-related-posts/) calculates the number of matched tags. Those are doable but I'm looking for more a content-based suggestion like LSI without manually generating a site. Until that, I think random posts could be okay in terms of exploring posts. Some might be actually relevant for readers.

{% highlight java %}
{% raw %}
{% assign posts = site.posts | where_exp:"post", "post.url != page.url" | sample:4 %}
{% for post in posts %}
  {% include related-post.html %}
{% endfor %}
{% endraw %}
{% endhighlight %}

> `site.posts` contains all posts including the current post. `where_exp` will prevent showing the current post as a related post.
