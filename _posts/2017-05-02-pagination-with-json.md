---
layout: post
title: "Pagination with JSON"
---

JSON(JavaScript Object Notation) format is a popular way to represent resources in order to communicate between a client and a backend. Compared to XML, it is more concise, compact and human-readable. JSON has two types: object and array. Array is the type to contain multiple objects. Depending on the number of resources you want to get, a different type could be utilized. Here are some backend endpoints.

* _/track/{trackId}_
* _/tracks/_

The first one is to get the single resource and the second one is to get collection resources. Because JSON format is used between a client and a backend, the efficiency is an important factor. It considers the amount of data transmitted, readability, flexibility. By considering the efficiency, there are several different ways to represent JSON format for the single resource and the collection resources.

## Naive Style
In this style, JSON object and JSON array are utilized directly.

<p class="code-label">/track/{trackId}</p>
{% highlight json %}
{
  "id": 1,
  "title": "Dancing Queen",
  "artist": "ABBA"
}
{% endhighlight %}


<p class="code-label">/tracks</p>
{% highlight json %}
[
  {
    "id": 1,
    "title": "Dancing Queen",
    "artist": "ABBA"
  },
  {
    "id": 2,
    "title": "SOS",
    "artist": "ABBA"
  },
  {
    "id": 3,
    "title": "Mamma Mia",
    "artist": "ABBA"
  }
]
{% endhighlight %}

One main advantage of this style is to use a small amount of data to transmit. However, this doesn't consider metadata including pagination for collection resources. The backend should return whole resources. You can easily imagine that it could be impossible once the number of resources is too many.

{% include google-adsense-in-article.html %}

## Envelope Style
JSON envelope is a wrapper with a named element. The previous style has some limitation for collection resources. To solve those, JSON envelope is used to wrap the data.

<p class="code-label">/track/{trackId}</p>
{% highlight json %}
{
  "id": 1,
  "title": "Dancing Queen",
  "artist": "ABBA"
}
{% endhighlight %}


<p class="code-label">/tracks</p>
{% highlight json %}
{
  "data": [
    {
      "id": 1,
      "title": "Dancing Queen",
      "artist": "ABBA"
    },
    {
      "id": 2,
      "title": "SOS",
      "artist": "ABBA"
    },
    {
      "id": 3,
      "title": "Mamma Mia",
      "artist": "ABBA"
    }
  ],
  "links": {
    "self": "/tracks?page=3&page_size=20",
    "first": "/tracks?page=1&page_size=20",
    "prev": "/tracks?page=2&page_size=20",
    "next": "/tracks?page=4&page_size=20",
    "last": "/tracks?page=9&page_size=20"
  },
  "meta": {
    "total_pages": 9
  }
}
{% endhighlight %}

With this small named element, now it is possible to send metadata including pagination.

## JSON API Style
So far, pagination is mainly considered so JSON envelope is applied to collection resources. Metadata is not only about the pagination. There could be some useful information not only for collection resources but also for the single resource. One meta example is a `share_url`. This style applies JSON envelope for both.

<p class="code-label">/track/{trackId}</p>
{% highlight json %}
[
  "data": {
    "id": 1,
    "title": "Dancing Queen",
    "artist": "ABBA"
  },
  "meta": {
    "share_url": "http://example.com/share/track/1"
  }
]
{% endhighlight %}

## Conclusion
There is no right and wrong style. It totally depends on your intent and data. What I want to highlight is that knowing different styles is important to avoid common mistakes. If you choose the naive style without knowing other styles, you may need to make another API endpoint to support the pagination later.

There could be more styles and there are many efforts to make efficient APIS in JSON format. [JSON API](http://jsonapi.org/format/) is one of them. Check their specification. You can use their library or you can learn many considerable points to make JSON format efficient.

## References
* [json:api](http://jsonapi.org/)
