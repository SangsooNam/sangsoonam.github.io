---
layout: post
title: "Picasso: Use Transformation, Not Target"
---

When you want to add some effects before showing an image, you need to access a bitmap in Android. One typical example is a blur effect. [Picasso](http://square.github.io/picasso/) is a image loading library. Generally, you load an image like this.

{% highlight java %}
Picasso.get()
       .load(imgaeUrl)
       .into(imageView);
{% endhighlight %}

## Target

When the image is loaded, it will be shown. You want to set some effects before showing it. So, you need to observe a lifecycle of an image loading in Picasso. Picasso has a custom `Target`. With that, you can listen to lifecycle events and do some actions. As you can see, `onBitmapLoaded` passes a loaded bitmap. Thus, you can make a blurred bitmap and set it into the image view.

{% highlight java %}
Picasso.get()
       .load(bigImageUrl)
       .into(new Target() {
            @Override
            public void onBitmapLoaded(Bitmap bitmap, Picasso.LoadedFrom from) {
                imageView.setImageBitmap(blurImage(bitmap));
            }

            @Override
            public void onBitmapFailed(Exception e, Drawable errorDrawable) {
                // Error handling
            }

            @Override
            public void onPrepareLoad(Drawable placeHolderDrawable) {
                imageView.setImageDrawable(placeHolderDrawable);
            }
      });
{% endhighlight %}

1. Although this works, I recommend not to use `Target` if your purpose is to transform an image before showing. Here are four reasons:

1. **Main thread**: `onBitmapLoaded` is called on the main thread. If your task is long or frequently called, your app would be slow or have a UI hang issue.

2. **Cache**: The loaded `bitmap` is cached by Picasso. However, Picasso doesn't care on a generated bitmap by you. Every time, there will be new created blurred bitmap though working a bitmap is expensive.

3. **Memory**: After creating a blurred bitmap, you cannot call `recycle` on the loaded bitmap. A bitmap is a quite object. As soon as possible, you need to call `recycle` to release memory. Since Picasso cares the loaded bitmap you cannot `recycle` it. There would be two bitmaps in the memory.

4. **Debug Indicators**: Picasso uses `PicassoDrawable` internally. It contains a logic to show debug indicators when you enable it. This is no real user impact but you will lose it when you handle `Target` directly.

{% include google-adsense-in-article.html %}

## Transformation

For transformations, Picasso supports a method, `transform(Transformation)` in the request chain. So, you need to create a class implementing `Transformation`. The result looks like this:

{% highlight java %}
Picasso.get()
       .load(imgaeUrl)
       .transform(new BlurTransformation(this))
       .into(imageView);
{% endhighlight %}

When you implement it, there are two override methods: `transform(Bitmap)` and `key()`. Unlike `onBitmapLoaded`, `transform` is called in Picasso thread. And, the returned value of `key()` is used for caching in Picasso. Picasso won't call `transform` if there is a cached bitmap. In addition, Picasso will force you to recycle the source bitmap before finishing `transform`. If you didn't recycle it, there will be an exception by Picasso. Lastly, Debug indicators work like before.

{% highlight java %}
public class BlurTransformation implements Transformation {

    private Context context;

    public BlurTransformation(Context context) {
        this.context = context;
    }

    @Override
    public Bitmap transform(Bitmap source) {
        Bitmap result = blurImage(source);
        // You need to recycle. Otherwise, Picasso will throw an exception.
        source.recycle();
        return result;
    }

    @Override
    public String key() {
        // This is the key used for caching.
        return "BlurTransformation";
    }
    ...
}
{% endhighlight %}

> Check the full [BlurTransformation.java](https://gist.github.com/SangsooNam/6c01f2932daf98df30f796d0de141444).

## Conclusion

You should utilize `transform()` when you want to transform an image rather than `Target`. With that, you don't need to worry about many things such as threading, caching and memory releasing. It is also possible to set multiple transformations.

{% highlight java %}
Picasso.get()
       .load(imgaeUrl)
       .transform(new BlurTransformation(this))
       .transform(new GrayscalTransformation())
       .into(imageView);
{% endhighlight %}

> If you are looking for different transformations, [Picasso Transformations](https://github.com/wasabeef/picasso-transformations) library provides a variety of image transformations for Picasso such as blur, crop, and color.
