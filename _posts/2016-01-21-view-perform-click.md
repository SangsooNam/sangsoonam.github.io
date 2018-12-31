---
layout: post
title: "Check before Calling performClick()"
---

![performClick](/images/2016/01-21/perform-click.png)

In Android, _View.performClick_ is defined as:

{% highlight java %}
/**
 * Call this view's OnClickListener, if it is defined.  Performs all normal
 * actions associated with clicking: reporting accessibility event, playing
 * a sound, etc.
 *
 * @return True there was an assigned OnClickListener that was called, false
 *         otherwise is returned.
 */
public boolean performClick()
{% endhighlight %}

Using calling method, view's _OnClickListener_ can be tested. More commonly, this is used to invoke view click through code. For example, a search button can be clicked when a enter key is entered.

{% highlight java %}
searchText.setOnKeyListener(new OnKeyListener() {
  @Override
    public boolean onKey(View v, int keyCode, KeyEvent event) {
      if (keyCode == KeyEvent.KEYCODE_ENTER && event.getAction() == KeyEvent.ACTION_UP ) {
        searchButton.perfomClick();
        return true;
      }
    return false;
  }
});
{% endhighlight %}

{% include google-adsense-in-article.html %}

If you use this method, it works mostly well like you expected. However, I've noticed that some people often have a problem because they forgot its definition. What do you expect about this example?

{% highlight java %}
searchButton.setOnClickListener(searchButtonClickListener);
searchButton.setEnabled(false);
searchButton.performClick();
{% endhighlight %}

_searchButton_ is disabled. Thus, you might think that _searchButtonClickListener_ is not called when _performClick_ method is called. Actually, it is called. How about this example?

{% highlight java %}
searchButton.setOnClickListener(searchButtonClickListener);
searchButton.setClickable(false);
searchButton.performClick();
{% endhighlight %}

This also calls _searchButtonClickListener_. Let's see the definition again. It says that this method calls view's _OnClickListener_. In short, it doesn't care the view's status. However, users will not reach _OnClickListener_ if a view is disabled(or not clickable).

You now understand the difference between _performClick_ and users click. If you call the _performClick_ in order to mimic a user's click, I suggest to use this helper method.

{% highlight java %}
public static boolean performClick(View view) {
  return view.isEnabled() && view.isClickable() && view.performClick();
}
{% endhighlight %}
