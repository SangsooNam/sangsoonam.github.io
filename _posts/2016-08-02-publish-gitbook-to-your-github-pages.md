---
layout: post
title: "Publish GitBook to Your GitHub Pages"
---

These days, I'm writing instruction manuals. After that, I would like to publish it as a website. So I've investigated about solutions, which suit below needs.

* Writing a document based on markdown syntax
* Embedding a video
* Generating a website
* Searching documents
* Custom theme

[GitBook](https://github.com/GitbookIO/gitbook) looks a nice option. See a generated site using it, ["Front-end Handbook"](https://www.gitbook.com/read/book/frontendmasters/front-end-handbook). It generates a static website from markdown documents. To install it, you just need to type one line command.

{% highlight sh %}
❯ npm install -g gitbook-cli
{% endhighlight %}

After that, you can initialize with `gitbook init` command. That will generate two files:_README.md_ and _SUMMARY.md_.

* _README.md_ is the first page and contains an introduction.
* _SUMMARY.md_ contains the table of content, which is shown as a left slide menu.

{% highlight sh %}
❯ gitbook init
info: create README.md
info: create SUMMARY.md
info: initialization is finished

❯ ls
README.md  SUMMARY.md
{% endhighlight %}

After modifying a document, run the `gitbook serve` command. If your document is modified, that will detect it and automatically update the static site.

{% highlight sh %}
❯ gitbook serve
Live reload server started on port: 35729
Press CTRL+C to quit ...

info: 7 plugins are installed
info: loading plugin "livereload"... OK
info: loading plugin "highlight"... OK
info: loading plugin "search"... OK
info: loading plugin "lunr"... OK
info: loading plugin "sharing"... OK
info: loading plugin "fontsettings"... OK
info: loading plugin "theme-default"... OK
info: found 1 pages
info: found 0 asset files
info: >> generation finished with success in 0.6s !

Starting server ...
Serving book on http://localhost:4000
{% endhighlight %}
> If you just want to build a static site only, run `gitbook buid` instead of `gitbook serve`.

{% include google-adsense-in-article.html %}

When you open the site (either open the `index.html` or `http://localhost:4000`), you will see a similar page like this.
![](/images/2016/08-02/gitbook.png)

Last but not least thing is publishing as a website. Github Pages allows us to publish a static website freely. What you need to do is to add files into the `gh-pages` branch. We already have generated files for that. So, remaining thing is to add those fils into the `gh-pages` branch. Unfortunately, `gitbook` doesn't support it as a command line option. You need to do it manually. To do it easily, I've created a shell script.

{% gist SangsooNam/aa73c3e1ff88d30433e4020f1275242a %}

Whenever you run this script, it will generate files for the static website and push into the `gh-pages` branch. With a continuous integration, such as TeamCity, it can be triggered hourly or when the repository file is changed.

{% highlight sh %}
❯ ./publish_gitbook.sh
{% endhighlight %}

> Your site will be available at `http(s)://<username>.github.io/<projectname>`.

If you want to use GitBook but are not interested in publishing your github pages, check [GitBook site](https://www.gitbook.com/). You can easily create manuals on the site with nice wysiwyg(what you see is what you get) editor.

## References

* [GitBook](https://www.gitbook.com/)
* [GitBook cli](https://github.com/GitbookIO/gitbook-cli)
* [Creating project pages](https://help.github.com/articles/creating-project-pages-manually)
