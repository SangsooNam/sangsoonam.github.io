---
layout: post
title: "Forward Compatibility with @JsonIgnoreProperties"
---

Most applications communicate with a backend using a JSON(JavaScript Object Notation). JSON is a lightweight and also human readable. After getting the data from the backend, you need to convert it to an object. By doing that, you don't need to handle the raw data format, JSON, anymore. There are several libraries to support this process, called deserialization. [Jackson](https://github.com/FasterXML/jackson) library is one of them.

When it deserializes JSON data, added annotations in the entity class are used. Thus, `email` and `name` value will be applied as constructor parameters to create a `User` object from JSON data.

<p class="code-label">JSON Data</p>
{% highlight json %}
{
  "email": "john@gmail.com",
  "name": "John Kevin"
}
{% endhighlight %}

<p class="code-label">Entity Class</p>
{% highlight java %}
public class User {

  public final String userEmail;
  public final String userName;

  public User(@JsonProperty("email") String userEmail,
              @JsonProperty("name") String userName) {
      this.userEmail = userEmail;
      this.userName = userName;
  }
}
{% endhighlight %}

<p class="code-label">Deserialization</p>
{% highlight java %}
ObjectMapper objectMapper = new ObjectMapper();
User user = objectMapper.readValue(jsonData, User.class);
{% endhighlight %}

Suppose you released this app with a version 1.0. Later, you want to show more user information in your app. To support it, the backend now returns more data.


<p class="code-label">JSON Data</p>
{% highlight json %}
{
  "email": "john@gmail.com",
  "name": "John Kevin",
  "gender": "M"
}
{% endhighlight %}

Then, the entity class has an additional constructor parameter with
`@JsonProperty("gender")`. `ObjectMapper` uses that key to get the data in the JSON data. Let's say this is the version 1.1. When you release this change to the market, some users will have a crash issue. Why some users rather than all users?

<p class="code-label">Entity Class</p>
{% highlight java %}
public class User {

  public final String userEmail;
  public final String userName;
  public final String userGender;

  public User(@JsonProperty("email") String userEmail,
              @JsonProperty("name") String userName,
              @JsonProperty("gender") String userGender) {
      this.userEmail = userEmail;
      this.userName = userName;
      this.userGender = userGender;
  }
}
{% endhighlight %}

{% include google-adsense-in-article.html %}

## UnrecognizedPropertyException

The crash happens because of `UnrecognizedPropertyException`. By default, Jackson generates this exception if there is unknown property. When the v1.0 was released, there was no exception. The issue started when the backend returned more data for v1.1. v1.1 doesn't have a problem because all properties from the backend JSON data are consumed. However, v1.0 doesn't know how to handle unrecognized property, `gender`. In short, only v1.0 has a crash issue. **It is vital to mention that not all users update their app up to date.**

Since users have an older version, the only way to fix it is to return the different data considering the app version from the backend. Although this would fix the crash issue, the backend code gets messy due to checking versions.

{% highlight java %}
if ("v1.1".equals(appVersion)) {
  ...
} else {
  ...
}
{% endhighlight %}

## @JsonIgnoreProperties

There is nothing we can do for already happened. However, we can prepare better if we know what could happen. I believe the best approach is to support the forward compatibility by adding `@JsonIgnoreProperties(ignoreUnknown = true)` always for the entity class. If the entity class in v1.0 had this annotation, there would have been no exception and no need to change the backend.

> _Forward compatibility or upward compatibility is a design characteristic that allows a system to accept input intended for a later version of itself. A standard supports forward compatibility if a product that complies with earlier versions can "gracefully" process input designed for later versions of the standard, ignoring new parts which it does not understand._
>
> _[Wikipedia](https://en.wikipedia.org/wiki/Forward_compatibility)_


<p class="code-label">Entity Class</p>
{% highlight java %}
@JsonIgnoreProperties(ignoreUnknown = true)
public class User {

  public final String userEmail;
  public final String userName;

  public User(@JsonProperty("email") String userEmail,
              @JsonProperty("name") String userName) {
      this.userEmail = userEmail;
      this.userName = userName;
  }
}
{% endhighlight %}

> There is another way to ignore properties. You can set `FAIL_ON_UNKNOWN_PROPERTIES` as false. However, the error still could happen. There could be multiple `ObjectMapper` and some may not have that configuration. Adding `@JsonIgnoreProperties` is safe since there is always one entity class.

<p class="code-label">Deserialization</p>
{% highlight java %}
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
User user = objectMapper.readValue(jsonData, User.class);
{% endhighlight %}

## Conclusion

In Jackson, `UnrecognizedPropertyException` happens because it doesn't support the forward compatibility by default. This type of error could be hard to know before experiencing. By adding  `@JsonIgnoreProperties(ignoreUnknown = true)` always in your entity class, your earlier apps can handle the later data gracefully.
