---
layout: post
title: "How to Set Up Picasso for Robolectric Test"
---

Running tests on an emulator or device is slow. For Android, [Robolectric](http://robolectric.org/) brings faster and reliable unit tests because it runs on the JVM. With Robolectric, you can test Android components such as `Activity` and `Fragment`. When you test those components, you need to consider something more because they are not a pure business logic class. For example, it could contain code for image loading using Picasso. Picasso is typically initialized in the `Application#onCreate`.

{% highlight java %}
public class YourApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        Picasso picasso = new Picasso.Builder(this).build();
        Picasso.setSingletonInstance(picasso);
    }
}
{% endhighlight %}

{% include google-adsense-in-article.html %}

Suppose you want to test `MainActivity` like below. `RobolectricTestRunner` is specified to use Robolectric.

{% highlight java %}
@RunWith(RobolectricTestRunner.class)
public class MainActivityTest {
    @Test
    public void testA() {...}

    @Test
    public void testB() {...}
}
{% endhighlight %}

When you run this, `IllegalStateException` happens from Picasso. The interesting thing is that `testA` would pass but `testB` would fail due to this exception. This is because `YourApplication` is reused for each test method. Instead of destroying `YourApplication`, lifecycle events(e.g. `onCreate` and `onDestroy`) are called before and after each test. Picasso doesn't allow to call `setSingletonInstance` twice so it throws an exception.

{% highlight java %}
java.lang.IllegalStateException: Singleton instance already exists.
	at com.squareup.picasso.Picasso.setSingletonInstance(Picasso.java:701)
{% endhighlight %}

**This only happens with Robolectric and never happens in your real device.** To resolve this issue, you can modify `YourApplication` to catch that exception and ignore it. Given that this doesn't happen in the production environment, catching exception sounds not great. The better way is to create an application only for Robolectric. You can specify it in `robolectric.properties`. When `RobolectricTestRunner` runs, it checks this file and that application is used for testing instead of `YourApplication`.

<p class="code-label">test/resources/robolectric.properties</p>
{% highlight ini %}
application=io.github.sangsoonam.RobolectricApplication
{% endhighlight %}

With a static boolean value, you can prevent a double calling on `setSingletonInstance`.

<p class="code-label">test/io/github/sangsoonam/RobolectricApplication.java</p>
{% highlight java %}
public class RobolectricApplication extends Application {
    private static boolean isPicassoInitialized;

    @Override
    public void onCreate() {
        super.onCreate();
        if (!isPicassoInitialized) {
            isPicassoInitialized = true;
            Picasso picasso = new Picasso.Builder(this).build();
            Picasso.setSingletonInstance(picasso);
        }
    }
}
{% endhighlight %}

Now, let's think about another point: network request. To show an image, that request is necessary. However, network request or disk access are not recommended while unit testing mainly for better performance. Mostly,  you don't consider image loading result. In that case, you can avoid the request by mocking Picasso.

<p class="code-label">test/io/github/sangsoonam/RobolectricApplication.java</p>
{% highlight java %}
Picasso picasso = Mockito.mock(Picasso.class, Answers.RETURNS_MOCKS);
Picasso.setSingletonInstance(picasso);
{% endhighlight %}

`Answers.RETURNS_MOCKS` makes you avoid having lots of mocking. Without that, you need to mock every method including nested calls.

{% highlight java %}
Picasso picasso = mock(Picasso.class);
RequestCreator requestCreator = mock(RequestCreator.class);
when(picasso.load(nullable(String.class))).thenReturn(requestCreator);
when(picasso.load(nullable(Uri.class))).thenReturn(requestCreator);
when(picasso.load(anyInt())).thenReturn(requestCreator);
...
{% endhighlight %}

Last but not least, actually you do not need to concern on the network request. Internally, Picasso uses `Dispatcher` and `DispatcherThread` to load an image. `DispatcherThread` has a looper to handle requests. Robolectric modifies or extends Android class behavior with Shadows. When you run with Robolectric, a looper doesn't process any queued actions unless you proceed manually. This is an example for the main looper.

{% highlight java %}
ShadowLooper.runMainLooperOneTask()
{% endhighlight %}


## Conclusion
Robolectric enables you to test Android components. If Picasso is used in that component, creating `RobolectricApplication` is a nice way to keep your production code clean and avoid `IllegalStateException`. With Robolectric, you don't need to worry about network request even if you use a real Picasso. If you prefer mocking, mock it with `Answers.RETURNS_MOCKS`.
