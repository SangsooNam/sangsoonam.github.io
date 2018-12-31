---
layout: post
title: "Failing Early for Null Pointer Exception"
---

NPE(Null Pointer Exception) is one of the common exceptions on Java development. It means that you are accessing some methods in an object which is `null`. NPE is a runtime exception. If it happens, your app will show that exception and crash.

In most case, this exception is somewhat unexpected when you develop your app. So, It is telling something is wrong. It is lucky to identify a problem point quickly. However, it is often hard to know the root cause since an object can be passed through multiple classes. To fix an issue quickly, we need to fail early. Here are some tips for handling with NPE.


## @NonNull and @Nullable
I highly recommend placing `@NonNull` and `@Nullable` as many places as you can. With those annotations, Android Studio can warn you early when you provide a wrong value.

![NonNull](/images/2018/12-30/nonnull.png)

`@NonNull`and `@Nullable` are powerful, but they have some limitations. You cannot modify third-party libraries or Android framework code. A warning doesn't work if there is a missing annotation. As you can see, `nullItem()` returns clearly `null` but Android Studio doesn't warn it anymore.

![NonNull](/images/2018/12-30/nonnull2.png)

Moreover, this check doesn't work on the runtime environment. Suppose the backend returns a `null` value after some changes. Your app could crash with NPE although a parameter has `@NonNull` annotation.

{% include google-adsense-in-article.html %}

## checkNotNull

To make sure you have a not null object while running the app, you need to verify objects, mostly at constructors and setters. Java supports assertions. You can verify a value or state by using `assert` keyword. Although it is supported by Java, I don't recommend it for Android development.

1. It requires an additional line. It makes code lengthy.
2. Assertions are unreliable in the Android runtime environment.

{% highlight java %}
public class User {

    @NonNull private String email;
    @NonNull private String name;

    public User(@NonNull String email, @NonNull String name) {
        assert email != null;
        assert name != null;

        this.email = email;
        this.name = name;
    }
    ...
{% endhighlight %}

![Assert](/images/2018/12-30/assert.png)

`checkNotNull` in [Guava library](https://github.com/google/guava) is a better way for both issues. It doesn't have any issues in the Android runtime. Moreover, it returns the input value after verifying it. We don't need an additional line anymore.

{% highlight java %}
import static com.google.common.base.Preconditions.checkNotNull;

public class User {

    @NonNull private String email;
    @NonNull private String name;

    public User(@NonNull String email, @NonNull String name) {
        this.email = checkNotNull(email);
        this.name = checkNotNull(name);
    }
    ...
{% endhighlight %}

## Conclusion

Failing early is important to fix NPE issues. `@NonNull` and `@Nullable` annotations would be helpful during development. That prevents putting a `null` value for `@NonNull` annotated parameter. For the runtime environment, use `checkNotNull` for constructors and setters. It will throw an exception immediately if that is `null`.
