---
layout: post
title: "Don't Use System.currentTimeMillis() for Time Interval"
---

Measuring time interval is implemented in many application. For example, Running application measures a runner's record. How can you implement it? In Android, `System.currentTimeMillis()` returns the difference, measured in milliseconds, between the current time and midnight, January 1, 1970 UTC. Using this method to measure the time interval sounds quite reasonable. So, you can simply think below implementation.



{% highlight java %}
// Store the start time
long startTime = System.currentTimeMillis();
...
// Calculate the time interval when the task is done
long timeInterval = System.currentTimeMillis() - startTime;
{% endhighlight %}

When you test this code, you can see that it works well. Then, why we need to avoid this method to measure time interval? The reason is related to "current time". This current time can be changed in two ways.

* **Manual change**: Users can change their time in the Setting page.
* **Automatic change**: Current time can be changed because of different time zones. If a user run across the time zone, the current time will be changed.

![TimeZone](/images/2017/03-01/timezone.png)

{% include google-adsense-in-article.html %}

To avoid those cases, we need to use another method. Android has a good utility class, `SystemClock`.  `SystemClock.elapsedRealtime()` is the method what we want. This returns elapsed time in milliseconds since the system was booted. This includes time spend in sleep such as CPU off, display dark, and etc.

> `SystemClock.uptimeMillis()` is counted in milliseconds since the system was booted. This looks quite similar with `SystemClock.elapsedRealtime()` but this clock stops when the system enters deep sleep.

To sum up, use `SystemClock.elapsedRealtime()` to measure the time interval always!

{% highlight java %}
// Store the start time
long startTime = SystemClock.elapsedRealtime();
...
// Calculate the time interval when the task is done
long timeInterval = SystemClock.elapsedRealtime() - startTime;
{% endhighlight %}

## References
* [System.currentTimeMillis()](https://developer.android.com/reference/java/lang/System.html#currentTimeMillis())
* [SystemClock.elapsedRealtime()](https://developer.android.com/reference/android/os/SystemClock.html#elapsedRealtime())
