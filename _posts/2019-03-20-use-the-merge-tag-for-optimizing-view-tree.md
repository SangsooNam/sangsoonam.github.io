---
layout: post
title: "Use the &lt;merge&gt; Tag for Optimizing View Tree"
---

In Android, a view is one of the basic blocks for user interface components. It is a tree structure and you can nest views to display what you want to show. If the tree height gets longer, it can lead to some performance issue. The reason is that the Android framework does a top-down traversal of the view tree two times: a measure pass and a layout pass. Thus, it is important to keep in mind to make the view tree height short as much as possible.

`FrameLayout` is one of the views, and it allows placements of views along the z-axis. So, you can stack view elements. In addition that, you can align views with a `layout_gravity` attribute. Here is an example.

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:textSize="20sp"
        android:text="Hello World!" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|center"
        android:text="Button" />

</FrameLayout>
{% endhighlight %}

{% include google-adsense-in-article.html %}

While running the application, you can capture the view tree. Click `Tools > Layout Inspector`. Then, you can see the below screen.

![layout-inspect-frame-layout](/images/2019/03-20/layout-inspect-frame-layout.png)

In the layout file, `FrameLayout` is the root view. That is true if you see that layout file. When you look closely at the view tree, you can find `ContentFrameLayout`, the parent of your `FrameLayout`. Android framework creates it internally and adds your views in it. It is a `FrameLayout` with a different name.

{% highlight java %}
public class ContentFrameLayout extends FrameLayout {
  public ContentFrameLayout(Context context) {
      this(context, null);
  }

  public ContentFrameLayout(Context context, AttributeSet attrs) {
      this(context, attrs, 0);
  }

  public ContentFrameLayout(Context context, AttributeSet attrs, int defStyleAttr) {
      super(context, attrs, defStyleAttr);
  }
}
{% endhighlight %}

If that is almost the same, `FrameLayout` in your layout file seems unnecessary. As you know, it is better to have a short view tree for the performance. You can do it with the `<merge>` tag. The `<merge>` tag is designed to eliminate redundant view groups. Android framework places views in the `<merge>` tag directly into the parent layout. The layout file can be changed like this.

 Since the parent layout is used, you can remove `layout_width` and `layout_height` attributes. Actually, any attributes in `<merge>` tag are just ignored even though you set.

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:textSize="20sp"
        android:text="Hello World!" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|center"
        android:text="Button" />

</merge>
{% endhighlight %}

![layout-inspect-content-frame-layout](/images/2019/03-20/layout-inspect-content-frame-layout.png)


## Conclusion

A shorter view tree brings better performance in Android. Android frame supports `<merge>` to eliminate redundancy view groups. It is useful especially when you use `FrameLayout` as a root view. With that, you can use the `ContentFrameLayout` directly.
