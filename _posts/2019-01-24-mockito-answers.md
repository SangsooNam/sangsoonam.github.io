---
layout: post
title: "Mockito Answers"
---

In [Mockito](https://site.mockito.org/), the most common way to create a mock object is to use either `@Mock` annotation or `Mockito.mock` method. When a method having a return value is called for a mock object, it returns an empty value such as `0`, empty collections, an empty string, and `null`.


{% highlight java %}
@Mock
private ConnectionInfo connectionInfo;

@Before
public void setUp() {
    MockitoAnnotations.initMocks(this);
}
{% endhighlight %}

{% highlight java %}
private ConnectionInfo connectionInfo;

@Before
public void setUp() {
    navigator = Mockito.mock(Navigator.class);
}
{% endhighlight %}

When you want to change a return value for testing, you can do it with `when-return`. This works well if the object's behavior is simple. However, there could be cumbersome work to write multiple `when-return`s if a mock object is too complex.

{% highlight java %}
when(connetionInfo.isConnected()).thenReturn(true);
{% endhighlight %}

For those cases, Mockito supports a way to set logic for specifying what to return instead of an empty value. You can do it by implementing  `Answer` interface and passing it when you mock an object. There are predefined `Answer`s for common cases in Mockito. You could utilize this rather than implementing a custom `Answer`. Here are some useful Mockito `Answer`s.

{% include google-adsense-in-article.html %}

## RETURNS_MOCKS

If a method return type is a custom class, a mock returns `null` because there is no empty value for a custom class. `RETURN_MOCKS` will try to return mocks if possible instead of `null`. Since `final` class cannot be mocked, `null` is still returned in that case.

{% highlight java %}
public class Album {
    @NonNull private String title;
    @NonNull private Image image;
    @NonNull private List<Track> tracks;

    public Album(@NonNull Image image,
                 @NonNull String title,
                 @NonNull List<Track> tracks) {
        this.image = image;
        this.title = title;
        this.tracks = tracks;
    }

    // Getters
    @NonNull
    public Image getImage() {
        return image;
    }
    ...
}
{% endhighlight %}

A below example verifies a behavior by changing the `getTitle()`. In `onDataLoaded()`, the presenter accesses `getImage().getUrl()` to show an image. Although `getTitle()` is a main condition to verify, there is an additional `when-return` to avoid a null pointer exception.

{% highlight java %}
@Test
public void test() {
    Album album = mock(Album.class);
    when(album.getTitle()).thenReturn("TITLE");
    when(album.getImage()).thenReturn(mock(Image.class)); // Note: Need to avoid NPE

    // This accesses `getImage().getUrl()`
    presenter.onDataLoaded(album);

    verify(viewBinder).showTitle("TITLE");
}
{% endhighlight %}

With `RETURNS_MOCKS`, this test could be simplified. You don't need to set up additional `when-return` since a mock is returned.

{% highlight java %}
@Test
public void test() {
    Album album = mock(Album.class, Answers.RETURNS_MOCKS);
    when(album.getTitle()).thenReturn("TITLE");

    presenter.onDataLoaded(album);

    verify(viewBinder).showTitle("TITLE");
}
{% endhighlight %}

When you use an annotation, you can specify `Answer` like below.

{% highlight java %}
@Mock(answer = Answers.RETURNS_MOCKS)
private Album album;
{% endhighlight %}


## RETURNS_SELF

This uses the return type of a method. If that type equals to the class or a superclass, it will return the mock. This is especially useful for builder classes. You don't need to implement `when-return` for builder methods.

{% highlight java %}
public  class HttpBuilder {

    private String uri;
    private List<String> headers;

    public HttpBuilder() {
        this.headers = new ArrayList<>();
    }

    public HttpBuilder withUrl(String uri) {
        this.uri = uri;
        return this;
    }

    public HttpBuilder withHeader(String header) {
        this.headers.add(header);
        return this;
    }

    public String request() {
        return uri + headers.toString();
    }
}

public class HttpRequesterWithHeaders {

    private HttpBuilder builder;

    public HttpRequesterWithHeaders(HttpBuilder builder) {
        this.builder = builder;
    }

    public String request(String uri) {
        return builder.withUrl(uri)
                .withHeader("Content-type: application/json")
                .withHeader("Authorization: Bearer")
                .request();
    }
}
{% endhighlight %}

You can see there are two `when-return`s to mock a builder properly.

{% highlight java %}
@Test
public void test() {
    HttpBuilder builder = mock(HttpBuilder.class);
    when(builder.withUrl(anyString())).thenReturn(builder); // Note: Need to avoid NPE
    when(builder.withHeader(anyString())).thenReturn(builder); // Note: Need to avoid NPE
    HttpRequesterWithHeaders requester = new HttpRequesterWithHeaders(builder);
    String request = "REQUEST_URI";

    when(builder.request()).thenReturn(request);

    assertThat(requester.request("URI")).isEqualTo(request);
}
{% endhighlight %}

When `RETURNS_SELF` is set, the mock is returned when it is assignable. Thus, you don't need to set up manually. Note that the mock is returned when the return type is `Object`. `Object` is a superclass of the class.

{% highlight java %}
@Test
public void test() {
    HttpBuilder builder = mock(HttpBuilder.class, Answers.RETURNS_SELF);
    HttpRequesterWithHeaders requester = new HttpRequesterWithHeaders(builder);
    String request = "REQUEST_URI";

    when(builder.request()).thenReturn(request);

    assertThat(requester.request("URI")).isEqualTo(request);
}
{% endhighlight %}

## RETURNS_DEEP_STUBS

When a class refers to another class, you sometimes have a chain of methods to get a value. This chain could be deep and multiple levels. For mocking them, you need to return a mock which returns a mock. For example, there are several `when-return`s because of a deep chain.

{% highlight java %}
@Test
public void test() {
    Artist artist = Mockito.mock(Artist.class);
    Album album = mock(Album.class);
    Track track = mock(Track.class);
    when(artist.getLatestAlbum()).thenReturn(album);
    when(album.getTracks()).thenReturn(Lists.newArrayList(track));
    when(track.getTitle()).thenReturn("TITLE");

    assertEquals("TITLE", artist.getLatestAlbum().getTracks().get(0).getTitle());
}
{% endhighlight %}

`RETURNS_DEEP_STUBS` makes it possible to set up a return value for a chain of methods without setting mocks between a chain. Thus, you can shortly set it like below.

{% highlight java %}
@Test
public void test() {
    Artist artist = Mockito.mock(Artist.class, Answers.RETURNS_DEEP_STUBS);
    when(artist.getLatestAlbum().getTracks().get(0).getTitle()).thenReturn("TITLE");

    assertEquals("TITLE", artist.getLatestAlbum().getTracks().get(0).getTitle());
}
{% endhighlight %}

## Conclusion

Mockito `Answer` supports a custom logic to return a value instead of a literal value. If you have multiple `when-then`s before verifying, consider using predefined Mockito `Answer`s. Otherwise, you can create your own `Answer` to simplify setup and focus on verification.
