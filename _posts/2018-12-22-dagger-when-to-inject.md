---
layout: post
title: "Dagger: When to Inject for Activity and Fragment"
---

On the dependency injection, it is preferred to inject as soon as possible so that we can avoid `NullPointerException`. For that reason, constructor injection is widely used. However, it is not easy to apply that rule for `Activity` and `Fragment`. The lifecycle of them is managed by the Android Framework. Furthermore, we don't call the `Activity`'s constructor directly when we create it.

{% highlight java %}
startActivity(new Intent(this, AnotherActivity.class));
{% endhighlight %}

For the dependency injection on `Activity` and `Fragment`, Dagger supports a helper method, `AndroidInjection.inject()`. Once it is called, dependencies will be injected. Since `onCreate` is the first callback of `Activity` lifecycle, it is the best place to inject for `Activity`.

When you inject, it is vital to call `AndroidInjection.inject()` before `super.onCreate()` although calling the super method first is common in other places. Dagger guide explains this:

> *It is crucial to call AndroidInjection.inject() before super.onCreate() in an Activity, since the call to super attaches Fragments from the previous activity instance during configuration change, which in turn injects the Fragments. In order for the Fragment injection to succeed, the Activity must already be injected.*
>
> *[Google Dagger: Dagger & Android](https://google.github.io/dagger/android#when-to-inject)*

{% highlight java %}
@Override
protected void onCreate(Bundle savedInstanceState) {
  // Note: Should call before `super.onCreate`
  AndroidInjection.inject(this);
  super.onCreate(savedInstanceState);
}
{% endhighlight %}

There is `onCreate` method in `Fragment` like `Activity`. You might think that is still the best place to inject. However, it is not true regarding on `Fragment`. If you check the [fragment lifecycle](https://developer.android.com/guide/components/fragments), there is another lifecycle callback method, `onAttach`, which is called before `onCreate`. Moreover, injecting at `onCreate` can make inconsistencies. If `Fragment` is reattached, `onCreate` is not called again at that time. So, we need to call `AndroidInjection.inject()` at `onAttach` for `Fragment`.

 {% highlight java %}
 @Override
 public void onAttach(Context context) {
   AndroidSupportInjection.inject(this);
   super.onAttach(context);
 }
 {% endhighlight %}

{% include google-adsense-in-article.html %}

## Conclusion

Dependencies should be injected as soon as possible. It is `onCreate` for `Activity` and `onAttach` for `Fragment`. When you call `AndroidInjection.inject()`, make sure to call it before its super method to avoid a wrong situation. If your class can extend `DaggerActivity` and `DaggerFragment`, consider to extend it because it already implemented this.

> If you use the support library fragment, use `AndroidSupportInjection.inject()` instead of `AndroidInjection.inject()`.

{% highlight java %}
public abstract class DaggerActivity extends Activity implements HasFragmentInjector {

  @Inject DispatchingAndroidInjector<Fragment> fragmentInjector;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    AndroidInjection.inject(this);
    super.onCreate(savedInstanceState);
  }

  @Override
  public AndroidInjector<Fragment> fragmentInjector() {
    return fragmentInjector;
  }
}
{% endhighlight %}

{% highlight java %}
public abstract class DaggerFragment extends Fragment implements HasFragmentInjector {

  @Inject DispatchingAndroidInjector<Fragment> childFragmentInjector;

  @Override
  public void onAttach(Context context) {
    AndroidInjection.inject(this);
    super.onAttach(context);
  }

  @Override
  public AndroidInjector<Fragment> fragmentInjector() {
    return childFragmentInjector;
  }
}

{% endhighlight %}
