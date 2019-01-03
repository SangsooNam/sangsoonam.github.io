---
layout: post
title: "Mockito: doReturn vs thenReturn"
---

In Mockito, you can specify what to return when a method is called. That makes unit testing easier because you don't have to change existing classes. Mockito supports two ways to do it: `when-thenReturn` and `doReturn-when`. In most cases, `when-thenReturn` is used and has better readability.

{% highlight java %}
User user = Mockito.mock(User.class);
when(user.getName()).thenReturn("John");
{% endhighlight %}

However, sometimes you need to consider using `doReturn-when` because there are different behaviors.

{% include google-adsense-in-article.html %}

## No type safety

The parameter of `doReturn` is `Object` unlike `thenReturn`. So, there is no type checking in the compile time. When the type is mismatched in the runtime, there would be an `WrongTypeOfReturnValue` execption.

{% highlight java %}
doReturn(true).when(user).getName());
{% endhighlight %}

This is the main reason why `when-thenReturn` is a better option if possible.


## No side effect

Although there is no type safety, the advantage of `doReturn-when` is no side effect. **In Mockito, a side effect happens to the real method invocation.** To understand this, you need to know when a real method invocation happens.

Generally, you first need to create a mock object with `Mockito.mock` before specifying a return value. When you call a method of the mock object, it returns a specified value but it doesn't do anything you defined in the class. There is no side effect so `when-thenReturn` and `doReturn-when` acts almost the same except `when-thenReturn` has a type safety.

{% highlight java %}
User user = Mockito.mock(User.class);
{% endhighlight %}

{% highlight java %}
when(user.getName()).thenReturn("John");
{% endhighlight %}

{% highlight java %}
doReturn(true).when(user).getName());
{% endhighlight %}

The difference comes when you create a spy with `Mockito.spy`. A spied object is linked to an actual object. So, there is a real method invocation when you call a method. This is valid even for when you do `when-thenReturn`. This is due to a parameter of `when` and it contains a way to invoke a method. e.g. `user.getName()`. In contrast, a parameter of `when` is only `user` when you use `doReturn-when`.

{% highlight java %}
User user = Mockito.spy(new User());
{% endhighlight %}

A side effect doesn't happen always but here are the usual cases:

1. A method throws an exception with precondition checking.
2. A method does what you don't want while unit testing. For example, network or disk access.

For example, `List#get()` throws an exception if the list is smaller. `doReturn` can specify a value without having an exception.

{% highlight java %}
List list = new LinkedList();
List spy = spy(list);

// Impossible: real method is called so spy.get(0)
// throws IndexOutOfBoundsException (the list is yet empty)
when(spy.get(0)).thenReturn("foo");

// You have to use doReturn() for stubbing
doReturn("foo").when(spy).get(0);
{% endhighlight %}

Another example is to change a class behavior for unit testing. `AccountService` is a target to test so you cannot mock it. With `Mocktio.spy` and `doReturn`(`doNothing` is for `void` return type), you can do unit testing without network access.

{% highlight java %}
public class AccountService extends IntentService {

    @Override
    protected void onHandleIntent(Intent intent) {
        if("DELETE".equals(intent.getAction())) {
            sendDeleteRequest();
        }
    }

    protected void sendDeleteRequest() {
        // Network access
        ...
    }
}
{% endhighlight %}

{% highlight java %}
@RunWith(RobolectricTestRunner.class)
public class AccountServiceTest {

    @Test
    public void shouldSendDeleteRequest_whenDeleteAction() {
        AccountService accountService = spy(new AccountService("Account"));
        doNothing().when(accountService).sendDeleteRequest();

        accountService.onHandleIntent(new Intent("DELETE"));

        verify(accountService).sendDeleteRequest();
    }
}
{% endhighlight %}

> If there is no side effect, you can still use `when-thenReturn` for a spied object.

{% highlight java %}
User user = Mockito.spy(new User());
when(user.getName()).thenReturn("John");
{% endhighlight %}

## Conclusion

You can use `doReturn-when` to specify a return value on a spied object without making a side effect. It is useful but should be used rarely. The more you have a better pattern such as MVP, MVVM and dependency injection, the less chance you need to use `Mockito.spy`. Then, you can use more type safe and readable `Mockito.mock`.
