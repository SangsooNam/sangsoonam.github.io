---
layout: post
title: "Your Presenter Might Know View Too Early"
---

When MVP(Model-View-Presenter) is applied in Android, the code is commonly written like below. `Contract.Presenter` is created in `onCreate()` and pass the _Activity_ or _Fragment_ implementing the `Contract.View`.

{% highlight java %}
public interface Contract {
  interface View {
    showLoading();
    showData(List<Data> list);
  }
  interface Presenter {
  }
}
{% endhighlight %}

{% highlight java %}
public class SearchActivity extends AppCompatActivity implements Contract.View {
  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate();
    mPresenter = new SearchPresenter(this);
  }
}
{% endhighlight %}

You can consider this code doesn't have any problem. However, it depends. According to the Android life cycle, Android `View` is not created yet. After `onViewCreated()` is called, we can sure it is created. If `Contract.Presenter` calls `showLoading()` before `onViewCreated()`, it could make a crash. So, I would recommend avoiding passing the `Contract.View` as a constructor parameter of `Contract.Presenter`.

Then, how can we set a `Contract.View` for `Contract.Presenter`? In the lifecycle, `View` is alive between `onStart()` and `onStop()`. By setting and unsetting the `Contract.View` at that time, we can avoid accessing `View` before created and after destroyed.

{% highlight java %}
public interface Contract {
  interface View {
    showLoading();
    showData(List<Data> list);
  }
  interface Presenter {
    void onViewAvailable(Contract.View view);
    void onViewUnavailable();
  }
}
{% endhighlight %}

{% highlight java %}
public class SearchActivity extends AppCompatActivity implements Contract.View {
  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate();
    mPresenter = new SearchPresenter();
  }

  @Override
  public void onStart() {
    super.onStart();
    mPresenter.onViewAttached(this);
  }

  @Override
  public void onStop() {
    super.onStop();
    mPresenter.onViewDetached();
  }
}
{% endhighlight %}

> Another approach is to keep the `Contract.View` as a constructor parameter but add `start()` and `stop()` method in `onStart()` and `onStop()`. The reason is that it can avoid the null check. I believe that is worse. `Contract.Presenter` should implicitly consider `start()` and `stop()` to call methods in `Contract.View`. If the `Contract.Presenter` is big, it is easy to make some mistakes. Furthermore, without `Contract.View` constructor parameter is more _Dagger_ friendly. You don't need to provide it :)

{% highlight java %}
public class SearchActivity extends AppCompatActivity implements Contract.View {
  @Inject SearchPresenter mPresenter;

  @Override
  public void onStart() {
    super.onStart();
    mPresenter.onViewAttached(this);
  }

  @Override
  public void onStop() {
    super.onStop();
    mPresenter.onViewDetached();
  }
}
{% endhighlight %}
