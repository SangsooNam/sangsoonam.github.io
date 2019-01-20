---
layout: post
title: "OkHttp: How to Refresh Access Token Efficiently"
---

When you use the token-based authentication including OAuth, there are two tokens: access token and refresh token. Whenever you need to access a protected resource, An access token should be used to approve the access right. A refresh Token is a kind of special token. It can be used to get a renewed access token because an access Token has a short lifecycle. The value of `expires_in` in the response shows how long the access token is valid.

{% highlight java %}
{
  "token_type":"bearer",
  "access_token":"{ACCESS_TOKEN_VALUE}",
  "expires_in":3600,
  "refresh_token":"{REFRESH_TOKEN_VALUE}"
}
{% endhighlight %}

Typically, an access token is used in the request header. OkHttp supports a handy way to tweak your request before sending it, which is `Interceptor`. With that, you can add an access token information in the header easily without modifying your request call methods.

{% highlight java %}
OkHttpClient client = okHttpClient.newBuilder()
  .addInterceptor(new AccessTokenInterceptor())
  .build();
{% endhighlight %}

{% highlight java %}
public class AccessTokenInterceptor implements Interceptor {

  private final AccessTokenRepository accessTokenRepository;

  public AccessTokenInterceptor(AccessTokenRepository accessTokenRepository) {
    this.accessTokenRepository = accessTokenRepository;
  }

  @Override
  public Response intercept(Chain chain) throws IOException {
    String accessToken = accountRepository.getAccessToken();
    Request request = newRequestWithAccessToken(chain.request(), accessToken);
    return chain.proceed(request);
  }

  @NonNull
  private Request newRequestWithAccessToken(@NonNull Request request, @NonNull String accessToken) {
    return request.newBuilder()
            .header("Authorization", "Bearer " + accessToken)
            .build();
  }
}
{% endhighlight %}

{% include google-adsense-in-article.html %}

This works while an access token is valid. Since the access token can be expired, you need to think how to refresh it. One naive approach could be to schedule a job to refresh before `expires_in`. It could work but there are things you need to consider.

* **Application Lifecycle**: When an application is stopped, a scheduled job could be gone. You would reschedule the job with considering the last refresh time when the application is started.

* **Unnecessary Refresh**: When a user doesn't access a protected resource for a long time, you don't need to refresh an access token. If there is a refreshing every 60 minute considering `expires_in`, there would be unnecessary refreshing 24 times for one day.

## Interceptor Approach
While intercepting, `Interceptor` allows you not only to modify your request but also to send a request and get a response. When an access token is expired, there is an HTTP unauthorized error, with 401 error code, from the backend. That is the best time to refresh an access token.

Below code shows how to do it. First, you need to send a refresh access token request. Then, retry the original request with the renewed access token. There are multiple threads in OkHttp to handle requests. It is important to use `synchronized` to avoid additional refreshing.


{% highlight java %}
public class AccessTokenInterceptor implements Interceptor {

  ...

  @Override
  public Response intercept(Chain chain) throws IOException {
    String accessToken = accessTokenRepository.getAccessToken();
    Request request = newRequestWithAccessToken(chain.request(), accessToken);
    Response response = chain.proceed(request);

    if (response.code() == HttpURLConnection.HTTP_UNAUTHORIZED) {
      synchronized (this) {
        final String newAccessToken = accessTokenRepository.getAccessToken();
        // Access token is refreshed in another thread.
        if (!accessToken.equals(newAccessToken)) {
            return chain.proceed(newRequestWithAccessToken(request, newAccessToken));
        }

        // Need to refresh an access token
        final String updatedAccessToken = accessTokenRepository.refreshAccessToken();
        // Retry the request
        return chain.proceed(newRequestWithAccessToken(request, updatedAccessToken));
      }
    }

    return response;
  }

  ...
}
{% endhighlight %}


## Authenticator Approach
`Interceptor` approach could be good enough to refresh an access token. Since the token-based authentication is getting common, OkHttp supports a better way, `Authenticator`. `Authenticator` is specially designed for refreshing an access token. There are three major benefits to use:

* **Auto Retry**: If you set a request, it will retry it up to 20 times by default.
* **No Response Code Check**: In the `Interceptor` approach, you manually check the response code to determine refreshing. `Authenticator` is only called when there is an HTTP unauthorized error, so you don't need to check.
* **No Interceptor Side Effect**: OkHttp can have multiple `Interceptor`s. Each interceptor is called orderly what you set and next `Interceptor` is executed when `chain.proceed()` is called. In `Interceptor` approach, there are two `chain.proceed()`. One is for the original request and the other is for retry request. In each time, the next `Interceptor` is executed, so `AnotherInterceptor` could be called twice. Depending on what it does, this could make a side effect.

`Authenticator` is triggered for every HTTP unauthorized error. For some authentication type such as `Basic`, it is not needed to refresh the access token. So, below code checks the header information first. The return type is `@Nullable`. If you return `null`, it doesn't do anything. Also, unlike `Interceptor`, it returns `Request` not `Response`.

{% highlight java %}
OkHttpClient client = okHttpClient.newBuilder()
  .authenticator(new AccessTokenAuthenticator())
  .addInterceptor(new AccessTokenInterceptor())
  .build();
{% endhighlight %}

{% highlight java %}
public class AccessTokenAuthenticator implements Authenticator {

  private final AccessTokenRepository accessTokenRepository;

  public AccessTokenAuthenticator(AccessTokenRepository accessTokenRepository) {
    this.accessTokenRepository = accessTokenRepository;
  }

  @Nullable
  @Override
  public Request authenticate(Route route, Response response) {
    final String accessToken = accountRepository.getAccessToken();
    if (!isRequestWithAccessToken(response) || accessToken == null) {
        return null;
    }
    synchronized (this) {
        final String newAccessToken = accountRepository.getAccessToken();
        // Access token is refreshed in another thread.
        if (!accessToken.equals(newAccessToken)) {
            return newRequestWithAccessToken(response.request(), newAccessToken);
        }

        // Need to refresh an access token
        final String updatedAccessToken = accountRepository.refreshAccessToken();
        return newRequestWithAccessToken(response.request(), updatedAccessToken);
    }
  }

  private boolean isRequestWithAccessToken(@NonNull Response response) {
      String header = response.request().header("Authorization");
      return header != null && header.startsWith("Bearer");
  }

  @NonNull
  private Request newRequestWithAccessToken(@NonNull Request request, @NonNull String accessToken) {
      return request.newBuilder()
              .header("Authorization", "Bearer " + accessToken)
              .build();
  }
}
{% endhighlight %}

## Conclusion
OkHttp supports token-based authentication well. `Interceptor` allows you to add the access token information in the header easily. For refreshing the access token, you can use `Authenticator`. It is specially designed for that and triggered only when there is an HTTP unauthorized error. Moreover, it will retry automatically and doesn't call `Interceptor` additionally.
