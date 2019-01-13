---
layout: post
title: "Share Your Bash Configuration for All Your Computers"
---

In the bash configuration (`.bash_profile` or `.bashrc`), you can set up useful things to help your command line interface. For example, `alias` and `function` can be set to make a shortcut command for a longer command. Environment variables are also configured so that other applications can refer a proper value.

<p class="code-label">.bash_profile</p>
{% highlight bash %}
# Alias
ll='ls -lh'

# Function
print-last-release() {
  git branch -r | grep release- | tail -1
}

# Environment variable
export ANDROID_HOME=/Users/sangsoonam/Library/Android/sdk/
{% endhighlight %}

When you have multiple computers, each computer will have own bash profile. Although this is reasonable, this is a bit cumbersome if you want to have a similar working environment for each computer and want to sync the configuration as much as possible. In my case, I often add some aliases at work. Then, I want to use them at home on my personal computer.

{% include google-adsense-in-article.html %}

There would be multiple ways to sync between them. Among them, it is quite easy to use Dropbox for syncing. Basically, you store your bash configuration in your Dropbox.

{% highlight bash %}
$ mv ~/.bash_profile ~/Dropbox/
{% endhighlight %}

Then, each computer uses the stored configuration with a symbolic link. Whenever there is a change, Dropbox will take care of syncing between computers.

{% highlight bash %}
$ ln -s ~/Dropbox/.bash_profile ~/.bash_profile
$ ls -al ~/.bash_profile
lrwxr-xr-x  1 sangsoonam  staff  29 Jan 13 16:07 .bash_profile -> /Users/sangsoonam/Dropbox/.bash_profile
{% endhighlight %}

## Conclusion
When you have several computers, you may want to sync some configuration files. You can store those files in Dropbox and link to point it. Dropbox will sync those files and help you to have the same working environment across computers. In addition to `.bash_profile` or `.bashrc`, `.gitconfig` could be good to sync because it contains specific aliases for `git`.

{% highlight bash %}
$ mv ~/.gitconfig ~/Dropbox/
$ ln -s ~/Dropbox/.gitconfig ~/.gitconfig
{% endhighlight %}

<p class="code-label">.gitconfig</p>
{% highlight ini %}
[alias]
  st = status
  cp = cherry-pick
  co = checkout
  subu = submodule update --init --recursive
  pp = !git pull upstream master && git subu
  clear = !git reset --hard && git clean -df
{% endhighlight %}
