---
layout: post
title: "Is 'TextUtils.isEmpty' Evil?"
---

While you are developing an application, we want to avoid duplicate logic as much as possible. For that, we usually make utility classes or apply existing third-party libraries.`TextUtils` is a utility class provided in the Android package. If it provides what you want, you don't need to make your own utility class or use other libraries. You can just use it in your code.

Among multiple methods in `TextUtils`, `TextUtils.isEmpty` is a common example. This method gets `CharSequence` parameter and returns `true` if the parameter is null or the length is 0. Although this is a quite simple method, this is commonly used in many places because it checks if the value is null.

<p class="code-label">android/text/TextUtils.java</p>
{% highlight java %}
public static boolean isEmpty(@Nullable CharSequence str) {
    return str == null || str.length() == 0;
}
{% endhighlight %}

Then, what is an issue of `TextUtils`? If you write a production code only, you don't see it at all. However, you will see it when you start to do a unit testing. Let's see it with a code example. In `onEmailTextChanged`, it uses `TextUtils.isEmpty` to determine whether to hide or show the next button. Of course, this code doesn't have any problem to run.

<p class="code-label">LoginPresenter.java</p>
{% highlight java %}
public void onEmailTextChanged(@Nullable String email) {
    if (TextUtils.isEmpty(email)) {
        mLoginView.hideNextButton();
    } else {
        mLoginView.showNextButton();
    }
}
{% endhighlight %}

For testing on `onEmailTextChanged`, we can consider three values: not null value, empty value, and null value. Here are tests to verify each behavior.

<p class="code-label">LoginPresenterTest.java</p>
{% highlight java %}
@Test
public void shouldShowNextButton_whenHaveEmail() {
    mPresenter.onEmailTextChanged("EMAIL");
    verify(mView).showNextButton();
}

@Test
public void shouldHideNextButton_whenEmptyEmail() {
    mPresenter.onEmailTextChanged("");
    verify(mView).hideNextButton();
}

@Test
public void shouldHideNextButton_whenNullEmail() {
    mPresenter.onEmailTextChanged(null);
    verify(mView).hideNextButton();
}
{% endhighlight %}

These tests are quite simple. However, there is an unexpected error when you run it. It is because of `TextUtils.isEmpty`.

{% highlight bash %}
java.lang.RuntimeException: Method isEmpty in android.text.TextUtils not mocked. See http://g.co/androidstudio/not-mocked for details.

  at android.text.TextUtils.isEmpty(TextUtils.java)
  at io.github.sangsoonam.textutils.LoginPresenter.onEmailTextChanged(LoginPresenter.java:14)
  at io.github.sangsoonam.textutils.LoginPresenterTest.shouldShowNextButton_whenHaveEmail(LoginPresenterTest.java:22)
{% endhighlight %}

Why does it happen? The short answer is that JUnit doesn't know Android related code at all. `TextUtils` is in the Android package. Since your application code will run on the Android device, your app will work completely well with `TextUtils`. However, JUnit and `TextUtils` live in a separate world. The key to solve this issue is how to link between them.

{% include google-adsense-in-article.html %}

## Solutions

There are mainly two ways.

1. **Use Java Library**:
This approach is not to use `TextUtils` at all. You can create a similar utility class or use third-party libraries. JUnit runs together with Java libraries so the problem will be gone.

2. **Use Robolectric**:
[Robolectric](http://robolectric.org/) is a framework that brings reliable unit tests with an Android environment. It doesn't require an Android emulator and devices.

Each approach has some pros and cons. If you create or use another Java library for that, you need to change the existing code. In addition, you can feel weird to change the code due to unit testing, although there is no problem to run.

How about using Robolectric? You don't need to change the code at all. Unlike JUnit, Robolectric will set up the Android environment so that `TextUtils` will work as you expected. You just need to add `@RunWith` to designate the test runner as Robolectric.

{% highlight java %}
@RunWith(RobolectricTestRunner.class)
public class LoginPresenterRobolectricTest {
{% endhighlight %}

This is straightforward. We've solved the issue, right? Well, yes but not optimally. Robolectric is not free. It will take time to set up the Android environment. Maybe you don't consider the unit test running performance. It actually gets important if you work together with other developers. In general setting, your pull request should pass all unit tests. Running unit tests is one of the top time-consuming tasks. I  want to highlight that Robolectric could be **10x slower** than normal JUnit.

![JUnit](/images/2018/12-02/junit.png)

![Robolectric](/images/2018/12-02/robolectric.png)

## Better Way

Let's see the example code again. You might notice that this code uses the MVP(Model-View-Presenter) pattern. Moreover, we try to test a presenter part where we don't want to make any dependency on Android. `TextUtils`, which has static methods, is an only one related to Android. Given that this code doesn't have a strong Android dependency, we can actually do some trick to keep the code and to bring the performance back.

`app/src/test` directory doesn't have only test classes. There could be some classes used in other cases. If we add `TextUtils` class in `app/src/test/android/text`, JUnit will use that class if the code uses `TextUtils`. I added that class with only one method, `isEmpty`, but you can add more methods that your code use.
> Check this repository to copy the code: [TextUtils.java](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/text/TextUtils.java)

<p class="code-label">android/text/TextUtils.java</p>
{% highlight java %}
package android.text;

public class TextUtils {
    public static boolean isEmpty(CharSequence str) {
        return str == null || str.length() == 0;
    }
}
{% endhighlight %}

## Conclusion

With this trick, you don't need to change the code and can have a faster unit testing. To sum up, I believe `TextUtils` is not evil. It just needs a little trick for a unit testing. Also, Robolectric is a great framework to test Android. I really love it when it is needed. We just need to consider more when to use it.

> You can check out [this repository](https://github.com/SangsooNam/textutils) to see the whole code and run it locally.
