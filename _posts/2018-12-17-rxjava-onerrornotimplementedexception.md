---
layout: post
title: "RxJava: OnErrorNotImplementedException"
---

[RxJava(Reactive Extensions for the JVM)](https://github.com/ReactiveX/RxJava) is a library for event-based programs. It allows composing asynchronous easily. _Publisher_ and _Consumer_ are main components. Literally, _Publisher_ will emit events and _Consumer_ will do some actions based on received events. Here is an example in Android:

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

This is a typical example of RxJava. However, I believe this is an anti-pattern. Your app could crash due to this code. When you have events or no event, it works fine. The problem occurs when Publisher emits errors. Without specifying an error consumer, the default,  `OnErrorMissingConsumer` will be set.

{% highlight java %}
public static final Consumer<Throwable> ON_ERROR_MISSING = new OnErrorMissingConsumer();
...
public final Disposable subscribe(Consumer<? super T> onNext) {
    return subscribe(onNext, Functions.ON_ERROR_MISSING, Functions.EMPTY_ACTION, Functions.emptyConsumer());
}
...
{% endhighlight %}

{% include google-adsense-in-article.html %}

For errors, `OnErrorMissingConsumer` makes a `OnErrorNotImplementedException`. This is `RuntimeException` and the app would crash unless you handled it. If you get the event or data from the network or database, there is a chance that you will get errors. It could be an internet connectivity problem or database locking. Although this makes the app crash, it could be hard to know this while developing. We usually don't test those error scenarios.

{% highlight java %}
public final class OnErrorNotImplementedException extends RuntimeException {
{% endhighlight %}

{% highlight bash %}
io.reactivex.exceptions.OnErrorNotImplementedException: Error
	at io.reactivex.internal.functions.Functions$OnErrorMissingConsumer.accept(Functions.java:704)
	at io.reactivex.internal.functions.Functions$OnErrorMissingConsumer.accept(Functions.java:701)
	at io.reactivex.internal.subscribers.LambdaSubscriber.onError(LambdaSubscriber.java:79)
	at io.reactivex.internal.subscriptions.EmptySubscription.error(EmptySubscription.java:54)
	at io.reactivex.internal.operators.flowable.FlowableError.subscribeActual(FlowableError.java:39)
{% endhighlight %}

This issue can be easily gone by specifying an error consumer. **Thus, I suggest you to set the error consumer to avoid the app crash. With the error consumer, you can guide users some actions or show proper messages.**

{% highlight java %}
getUser() // Publisher. Events could be from the network or database.
  .subscribeOn(Schedulers.io())
  .observeOn(AndroidSchedulers.mainThread())
  .subscribe(user ->  {
      userTextView.setText(user.getName());
    }, throwable -> {
      // Error Consumer
      userTextView.setText("Unknown");
    }
  });
{% endhighlight %}
