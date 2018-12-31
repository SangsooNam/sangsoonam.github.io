---
layout: post
title: "Live Templates for Annotators"
---

![Live Templates](/images/2016/02-12/live-templates.png)

An annotation is a form of syntactic metadata which can be added into Java source code. Using it, you can add some additional information for parameters, methods and classes. Personally, I use `@NonNull` and `@Nullable` annotations a lot. With them, I know a parameter requirement and a mismatched type easily.

{% highlight java %}
public Image(@NonNull String url, @NonNull String title);
{% endhighlight %}

{% highlight java %}
/**
 * A title in the Image constructor is @NonNull but
 * a title parameter here is @Nullable. There is a mismatched type
 */
public static Image createImage(@NonNull String url, @Nullable String title) {
  return new Image(url, title);
}
public Image(@NonNull String url, @NonNull String title) {

}
{% endhighlight %}

{% include google-adsense-in-article.html %}

There are really good annotations. However, I've noticed that two things.

1. Typing `@NonNull` and `@Nullable` is not that short.
2. After typing it, I often need to import the package.

I need to find a better way. Fortunately, _Live Templates_ saves me. You can see how I do it now. I just type `nu<space>` and `nn<space>`. This is not only about replacing the text but also about importing the package.

![NonNull and Nullable](/images/2016/02-12/nonnull_and_nullable.gif)

To do it, open _Preference >  Live Templates_. Then,

1. Click '+' to add a new one
2. Type `nn` in _Abbreviation_
3. Type `@android.support.annotation.NonNull ` (There is a space at the end) in _Template text_
4. Select 'Space' in _Expand with_
5. Check 'Use static import if possible'
6. Define applicable contexts as 'java > expression' and 'java > declaration'

![NonNull Setting](/images/2016/02-12/nonnull_setting.png)

This is a setting for `@Nullable`.
![Nullable Setting](/images/2016/02-12/nullable_setting.png)

> If you want to know more advanced live templates, you can see my live template for the argument captor.
> ![Argument Captor](/images/2016/02-12/argument_captor.gif)
> `$1$` and `$2$` are variables. For this, I don't check 'Use static import if possible'
> ![Argument Captor Setting](/images/2016/02-12/argument_captor_setting.png)
