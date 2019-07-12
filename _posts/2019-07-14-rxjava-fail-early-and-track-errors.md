---
layout: post
title: "RxJava: Fail Early and Track Errors"
---

In RxJava, _Publisher_ emits events and you can subscribe to those events with _Consumer_. While subscribing, emitted events can be used to update the view or to do something else. Below example shows a simple example:

{% highlight java %}
getUser() // Publisher. Events could be from the network or database.
  .subscribeOn(Schedulers.io())
  .observeOn(AndroidSchedulers.mainThread())
  .subscribe(user ->  {
      // Consumer
      userTextView.setText(user.getName());
    }
  });
{% endhighlight %}

This code work well while _Publisher_ emits events without any error. However, there could be several issues to emit events. Those vary depending on where events are generated. For example, if _Publisher_ is related to the network, it could have a bad connection or no response from a backend service. In that situation, an error will be thrown. The current code doesn't have any logic to consume that error, `OnErrorMissingConsumer` exception happens. That exception makes an app crash.

{% include google-adsense-in-article.html %}

The crash will tell you what went wrong. Although it is useful, you don't want to make a crash in some cases. Some logic doesn't affect the main experience, or a user can trigger to subscribe again easily. So, you can set an error consumer to handle those errors and avoid a crash. One way is just to print an error log.

{% highlight java %}
getUser() // Publisher. Events could be from the network or database.
  .subscribeOn(Schedulers.io())
  .observeOn(AndroidSchedulers.mainThread())
  .subscribe(user ->  {
      // Event Consumer
      userTextView.setText(user.getName());
    }, throwable -> {
      // Error Consumer
      Log.e(TAG, "Failed to get the user name");
    }
  });
{% endhighlight %}

It looks good, but it has some potential issues.

1. This doesn't make a crash, so it is easy to miss. Error logs are often not enough to grab attention.
2. There is no way to know how many people have errors. If your backend service is down, all users will have a network error. By tracking errors, you can keep giving a better experience continuously.

Thus, what we want to achieve is to track errors in the release and to make a crash while developing, which helps early detection. _Crashlytics_ is one of the useful tools for tracking errors. You can track errors by calling `Crashlytics.logException`.

{% highlight java %}
Crashlytics.logException(throwable);
{% endhighlight %}

`BuildConfig.DEBUG` tells whether the build is a release or a debug. TO sum up, you can resolve those issues like below.

{% highlight java %}
...
    }, throwable -> {
      // Error Consumer
      RxJavaErrorHandler.handle(throwable);
    }
...
{% endhighlight %}

{% highlight java %}
public class RxJavaErrorHandler {

    public static void handle(Throwable throwable) {
        if (BuildConfig.DEBUG) {
            Thread.currentThread()
                    .getUncaughtExceptionHandler()
                    .uncaughtException(Thread.currentThread(), throwable);
        } else {s

            Crashlytics.logException(throwable);
        }
    }
}
{% endhighlight %}

> Some exceptions/throwables could be not needed to make a crash in your application. You can check with `instanceof` and ignore them.
