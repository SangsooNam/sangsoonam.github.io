---
layout: post
title: "Realtime Stopwatch"
---

Stopwatch is an application which users allow to measure the elapsed time between the start and the stop.

![Stopwatch](/images/2017/07-10/stopwatch.png)

Here are general requirements for that.

* Have buttons to start, stop.
* When it is started, **ticking time** should be displayed.
* Ticking time should be shonw in the **millisecond precision**.
* When it is stopped, elapsed time should be displayed.

Requirements looks not difficult. However, if you want to make this application, you need to consider how often update the view to display the ticking time. If the view is updated too frequently, it will use the device resource a lot but people may not recognize. Let's try to update it as soon as possible. Since the view should be updated by UIThread, the simple way is to call `View#post`.

{% highlight java %}
private final Runnable mUpdateTextView = new Runnable() {
    @Override
    public void run() {
        mTextView.setText(String.valueOf(System.currentTimeMillis() - mStartTime));
        mTextView.post(mUpdateTextView); // Call post again to update the time
    }
};

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    mStartStopButton = (Button) findViewById(R.id.btn_start_stop);
    mTextView = (TextView) findViewById(R.id.txt_time);
    mStartStopButton.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            mStarted = !mStarted;
            mStartStopButton.setText(mStarted ? R.string.btn_stop: R.string.btn_start);

            if (mStarted) {
                mStartTime = System.currentTimeMillis();
                mTextView.post(mUpdateTextView);
            } else {
                mTextView.removeCallbacks(mUpdateTextView);
            }
        }
    });
}
{% endhighlight %}

{% include google-adsense-in-article.html %}

This works well although it seems to call `post` too many. There is no delay. We all know human can see more or less 60 frames per second. Given that fact, how about adding a delay (1000 / 60 = 16.6 millis)?

{% highlight java %}
private final Runnable mUpdateTextView = new Runnable() {
...
        mTextView.postDelayed(mUpdateTextView, 16); // Call post again to update the time
...
};

...
@Override
protected void onCreate(Bundle savedInstanceState) {
...
                mTextView.postDelayed(mUpdateTextView, 16);
...
}
{% endhighlight %}

This works smoothly and sounds better than without a delay. In fact..., there is no big difference.

 [`systrace`](https://developer.android.com/studio/profile/systrace.html) is a great tool to compare the performance. It supports a custom section so that you can put it where you want to check. Below shows two custom sections in `Runnable#run` for no-delayed and delayed one.

{% highlight java %}
@Override
private final Runnable mUpdateTextView = new Runnable() {
    @Override
    public void run() {
        Trace.beginSection("UPDATE");
        mTextView.setText(String.valueOf(System.currentTimeMillis() - mStartTime));
        mTextView.post(mUpdateTextView); // Call post again to update the time
        Trace.endSection();
    }
};
{% endhighlight %}

{% highlight java %}
@Override
private final Runnable mUpdateTextView = new Runnable() {
    @Override
    public void run() {
        Trace.beginSection("UPDATE_16");
        mTextView.setText(String.valueOf(System.currentTimeMillis() - mStartTime));
        mTextView.postDelayed(mUpdateTextView, 16); // Call post again to update the time
        Trace.endSection();
    }
};
{% endhighlight %}

Then, you can run `systrace.py` to get the trace report.

> It is imporatnt to put your application, `--app=...`. Otherwise, you won't see custom sections.

{% highlight sh %}
$ cd android-sdk/platform-tools/systrace
$ python systrace.py --time=10 -o trace.html gfx view wm --app=io.github.sangsoonam.realtime_stopwatch
{% endhighlight %}


_No-delayed_ ([Full trace report](/images/2017/07-10/trace.html))
![Stopwatch](/images/2017/07-10/trace_update_anno.png)

_Delayed_ ([Full trace report](/images/2017/07-10/trace_16.html))
![Stopwatch](/images/2017/07-10/trace_update_16_anno.png)

Method calling time is slightly different because of the delay. However, you can see that it is called in between each frame and it has a similar interval. Why this happens?

The main reason is that `View#post` doesn't execute a `Runnable` immediately. It causes the `Runnable` to be added to the message queue. That is periodically checked and executed right after frame time. Frame interval and delay was almost the same. Consequently, there was a big different method calling time but calling interval was quite similar with and without a delay.

To be sum up, you can use the `View#post` without any delay if you want to update the view really frequently. That already considers a frame time.

> Check this repository and run it if you want to check by yourself.
> [https://github.com/SangsooNam/realtime-stopwatch](https://github.com/SangsooNam/realtime-stopwatch)
