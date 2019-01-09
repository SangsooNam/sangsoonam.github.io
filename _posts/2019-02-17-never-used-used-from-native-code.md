---
layout: post
title: "Never Used? @UsedFromNativeCode"
---

Android NDK(Native Development Kit) is a tool that allows you to implement parts of your application in native code, using C and C++ language. The main reason to use native code is performance. Native code is complied to a binary and it runs directly on OS. In Android, Java code is translated into Java byte-code. Unlike native code, it is run by Dalvik virtual machine, not OS.

Native code and Java code should communicate at some point. Since they are using different languages, it is quite interesting. When native code wants to send a data to Java code, it gets a Java method id based on its name and signature. Then, it calls that method along with parameter values.

{% highlight c %}
mid = getMethodId(env, obj, "onLogin", "(Ljava/lang/String;Ljava/lang/String;)V");
{% endhighlight %}

{% highlight java %}
public class NativeCallback {

    void onLogin(String userId, String token) {
        ...
    }

    void onSessionEvent(int event, int param) {
        ...
    }
}
{% endhighlight %}

Android Studio is getting better on supporting native code. However, it seems hard to understand whether a method is used in native code. Although a method is used in the native code, Android Studio warns you to remove an unused declaration.

![Never used](/images/2019/02-17/never-used.png)

{% include google-adsense-in-article.html %}

## @SuppressWarnings

Without knowing the native code side, other engineers will remove that method without any doubt. If you suppress the warning with `@SuppressWarnings`, Android Studio doesn't suggest removing a method anymore. This would make a slightly better situation.

![@SuppressWarnings](/images/2019/02-17/suppress-warnings.png)

However, it still doesn't give enough information for other engineers. If they change the method name and its signature, native code will fail to find a Java method. There should be a better way to mention a relationship to native code.

## @UsedFromNativeCode

For that, you can add a custom annotation, `@UsedFromNativeCode`.

{% highlight java %}
package io.github.sangooonam;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

/**
 * This annotation is used to indicate that a method is used from native code. It is mostly
 * used along with @SuppressWarnings("UnusedDeclaration") to clarify why it appears unused in IDE.
 * If a method has this annotation, developers should have extra caution while refactoring
 * such as renaming and changing a signature.
 */
@Retention(RetentionPolicy.CLASS)
public @interface UsedFromNativeCode {
}
{% endhighlight %}

This annotation makes clear whether a method is used from native code.

{% highlight java %}
public class NativeCallback {

    @UsedFromNativeCode
    @SuppressWarnings("UnusedDeclaration")
    void onLogin(String userId, String token) {
        ...
    }

    @UsedFromNativeCode
    @SuppressWarnings("UnusedDeclaration")
    void onSessionEvent(int event, int param) {
        ...
    }
}
{% endhighlight %}

`@SuppressWarnings` is still useful but it looks a bit verbose now. Android Studio supports to suppress a warning if there is a certain annotation. That makes you suppress an unused warning without `@SuppressWarnings` annotation.

![Suppress warning annotated by](/images/2019/02-17/suppress-warning-annotated-by.png)


## Progurad

`@UsedFromNativeCode` helps not only maintainability but also a ProGuard configuration. Native code refers to a Java method using a method name. The method name should not be obfuscated and the method itself should not be removed by ProGuard. Typically, you need to write down each class and method that you want to keep in ProGuard configuration. Since they now have `@UsedFromNativeCode` annotation, you can set below configuration. This will work for all annotated methods.

{% highlight nim %}
-keep,allowobfuscation @interface io.github.sangsoonam.UsedFromNativeCode
-keepclassmembers class * {
    @io.github.sangsoonam.UsedFromNativeCode *;
}
{% endhighlight %}

![ProGuard](/images/2019/02-17/proguard.png)

## Conclusion

Native code and Java code use different languages. Android Studio doesn't detect well whether a Java method is used in native code. A custom annotation can help this issue. With `@UsedFromNativeCode`, you can clarify where the method is used and can suppress unused warnings effectively. Moreover, it makes ProGuard configuration short and easy.
