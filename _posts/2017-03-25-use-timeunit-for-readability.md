---
layout: post
title: "Use TimeUnit for Readability"
---

One day is 24 hours. One hour is 60 minutes. One minute is 60 seconds, and One second is 1000 milliseconds. It is straightforward. Then, can you easily say hours from below milliseconds?

{% highlight java %}
86,400,000
{% endhighlight %}

If this is a bit hard, how about this form?

{% highlight java %}
1000 * 60 * 60 * 24
{% endhighlight %}

`1000 * 60` is one minute. and `one minute * 60` is one hour. So, the answer is 24 hours, `one hour * 24`. Latter form could be easier but both are not human-readable.

In programming, millisecond is a common time unit. It has an enough precision for most cases, especially for scheduling and measuring time durations. Consequently, you can easily see those magic numbers (e.g. `86,400,000`). Those are relevant numbers but just not easy to read/understand immediately for humans.

`TimeUnit` was introduced to mitigate this issue. This is in `java.util.concurrent`. This is a `Enum` class and has following constants: `DAYS`, `HOURS`, `MICROSECONDS`, `MILLISECONDS`, `MINUTES`, `NANOSECONDS`, `SECONDS`. With these constants, now you can write much human-readable code like below.

{% highlight java %}
System.out.println(86400000);
System.out.println(TimeUnit.DAYS.toMillis(1)); // Result: 86400000
System.out.println(TimeUnit.HOURS.toMillis(24)); // Result: 86400000
System.out.println(TimeUnit.MILLISECONDS.toDays(86400000)); // Result: 1
{% endhighlight %}

{% include google-adsense-in-article.html %}

In addition, `TimeUnit` supports a `sleep` method. You probably have seen this code snippet.

{% highlight java %}
try {
  Thread.sleep(4000);
} catch (InterruptedException e) {
  e.printStackTrace();
}
{% endhighlight %}

With `TimeUnit`, this code can be written like this.
{% highlight java %}
try {
  TimeUnit.SECONDS.sleep(4);
} catch (InterruptedException e) {
  e.printStackTrace();
}
{% endhighlight %}

Of course, you can put `TimeUnit.SECONDS.toMillis(4)` as a paremeter of `Thread.sleep`. `TimeUnit#sleep` is simply a convenient way to do.

To sum up, `TimeUnit` gives us a big readability improvement. The more readability we have, the less we have bugs due to misunderstanding.

## References
* [TimeUnit](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/TimeUnit.html)
