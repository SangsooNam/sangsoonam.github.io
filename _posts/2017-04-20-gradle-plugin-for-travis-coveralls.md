---
layout: post
title: "Gradle Plugin for Travis and Coveralls"
---

![](/images/2017/04-20/main.png)

If you think to make an open source project, [Travis CI](https://travis-ci.org/) is a good choice for the continuous integration. It is well known and documented. Moreover, there is no charge for open source projects :). With the continuous integration, you can check easily whether your build is broken or not after you submit a commit or merge a pull request. This is a good and I think having a test coverage is much better. It can show that you care about the code quality well for others. [Coveralls](https://coveralls.io) supports a test coverage history and statistics. This is also free for open source projects. So, using both is a  wonderful combination for open source projects.

The way to integrate both depends on your project tools. I'm using [Gradle](https://gradle.org/) actively in these days so I've checked how to do it with Gradle. Gradle has plugins which are reusable pieces of build logic. I find this plugin,  [coveralls-gradle-plugin](https://github.com/kt3k/coveralls-gradle-plugin).

With this plugin, it is quite easy to integrate. You need add that plugin and set `xml.enabled` as `true` which is needed for Coveralls.

{% include google-adsense-in-article.html %}

<p class="code-label">build.gradle</p>
{% highlight groovy %}
plugins {
    id 'jacoco'
    id 'com.github.kt3k.coveralls' version '2.6.3'
}

jacocoTestReport {
    reports {
        xml.enabled = true // coveralls plugin depends on xml format report
        html.enabled = true
    }
}
{% endhighlight %}

Then, set below to call `jacocoTestport` and `coveralls` tasks in Travis.

<p class="code-label">.travis.yml</p>
{% highlight yaml %}
after_success:
- ./gradlew test jacocoTestReport coveralls
{% endhighlight %}

That's it. Once Travis runs, you can see new build request in Coveralls.

![](/images/2017/04-20/travis-coveralls.png)


> When you add your repository into Coveralls, you can see something about `repo_token`. I think this is not clear enough. You don't need to do anything if your project is an open source project. Everything will work well in Travis. However, If you run the `coveralls` task in your local machine instead of Travis, set an environment variable like this.
{% highlight bash %}
export COVERALLS_REPO_TOKEN=<REPO_TOKEN>
{% endhighlight %}

## References
* [Travis CI](https://travis-ci.org/)
* [Coveralls](https://coveralls.io)
* [Gradle](https://gradle.org/)
* [Coveralls Gradle Plugin](https://github.com/kt3k/coveralls-gradle-plugin)
