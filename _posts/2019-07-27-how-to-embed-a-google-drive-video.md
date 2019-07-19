---
layout: post
title: "How to Embed a Google Drive Video on a Website"
---

There are lots of video sites you can upload your videos. After uploading, you can easily embed that video on your post or on a website. For example, YouTube supports _"Copy embed code"_ option when you do the right-click.

![YouTube embed code](/images/2019/07-27/youtube-embed-code.png)

The copied code is like below. `iframe` tag is used to embed a video, and there are other parameters such as `allowfullscreen`.

{% highlight html %}
<iframe width="766" height="431" src="https://www.youtube.com/embed/hVGVcmW0crs"
frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope;
picture-in-picture" allowfullscreen></iframe>
{% endhighlight %}

{% include google-adsense-in-article.html %}

As you can see, it is easy to embed videos on your site. Although there are well-known services, you might consider embedding a Google drive video rather than using them. There are several benefits. When you upload your video, most sites consider that as public. It means that you might not be able to specify who you want to show. When you share something from Google drive, you can easily set who can see. It also has a handy permission scope, an organization.

That being said, embedding a Google drive video supports a better permission scope and security. If you consider it, here are the steps. After uploading your video, you first need to share it. You can set specific users or an organization.

![Google drive share](/images/2019/07-27/gdrive-share.png)

Unlike YouTube, finding _"Embed code"_ in Google drive is not straightforward. First of all, you need to click _"Preview"_.
![Google drive preview](/images/2019/07-27/gdrive-preview.png)

Then, open the menu by pressing the top-right icon, and click _"Open in new window"_. That will open a new tab.

![Google drive preview](/images/2019/07-27/gdrive-open-in-new-window.png)

When you open the menu again in that tab, you will see the option, _"Embed item..."_.
![Google drive embed item](/images/2019/07-27/gdrive-embed-item.png)

Finally, you can copy the embed code and paste it on your site.
![Google drive embed item code](/images/2019/07-27/gdrive-embed-item-code.png)

Note that full screen mode is not allowed if you just copy and paste that code. You can allow it by adding `allowfullscreen` in the `iframe` tag.

![Google drive full screen](/images/2019/07-27/gdrive-full-screen.png)

{% highlight html %}
<iframe src="https://drive.google.com/file/d/1rLT6EI-example/preview"
width="640" height="480" allowfullscreen></iframe>
{% endhighlight %}
