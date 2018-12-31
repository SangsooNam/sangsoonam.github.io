---
layout: post
title: "No Gradle Daemon for CI Agents"
---

Gradle runs on the JVM with several supporting libraries that require an initialization time. If you run Gradle tasks frequently,  it would be better to minimize duplicate initialization time. Gradle Daemon is the solution for that. It is a long-lived background process that executes tasks quickly if possible. It avoids expensive bootstrapping process and leverages caching. So, there is no reason not to utilize Gradle Daemon.

In the local environment, it is true. However, it could be wrong if you use CI agents such as Jenkins and TeamCity. CI agents could be reused by different projects or tasks. You should make sure that agent is reusable after using it. Otherwise, CI agents could fail to run tasks occasionally. As I mentioned, Gradle Daemon is a long-lived backend process. It could be still running after finishing your tasks along with holding some resources. `LockTimeoutException` is one of the examples that another task fails due to the Gradle Daemon.

{% highlight bash %}
Caused by: org.gradle.cache.LockTimeoutException:
  Timeout waiting to lock journal cache (/home/user/.gradle/caches/xxx-1).
  It is currently in use by another Gradle instance.
{% endhighlight %}

{% include google-adsense-in-article.html %}

To resolve this issue, you shouldn't use Gradle Daemon. There are two ways to set not to use it. One is to set it in `gradle.properties`. If you don't have `org.gradle.daemon` key, it equals to `false`. The other way is to add `--no-daemon` when you run `./gradlew`. Your tasks will be executed without Gradle Daemon regardless of the value in the `gradle.properties`.

<p class="code-label">gradle.properties</p>
{% highlight properties %}
org.gradle.daemon=false
{% endhighlight %}

{% highlight bash %}
$ ./gradlew --no-daemon build
{% endhighlight %}

If you set it up correctly, you should see below message when you run your tasks.

{% highlight bash %}
$ ./gradlew --no-daemon build
  To honour the JVM settings for this build a new JVM will be forked.
  Please consider using the daemon: https://docs.gradle.org/4.10.2/userguide/gradle_daemon.html.
{% endhighlight %}

Between two ways, I recommend using `--no-daemon` way and keeping `true` in `gradle.properties`. Since `gradle.properties` is commonly shared through the git, you and your coworkers will keep the performance improvement in the local environment. CI agents will be safer with `--no-daemon`.
