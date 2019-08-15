---
layout: post
title: "Failing Unit Test by GeneratedSerializationConstructorAccessor1"
---

For running Gradle tasks, I prefer using a command-line interface. Recently, I made a code change and added new unit tests. To verify the behavior, I ran a `test` task. Soon after, it showed a quite weird error message, which was about _GeneratedSerializationConstructorAccessor1_. As you can see below, the error message didn't point out any line from my code. Everything was related to `jdk`. When I made a pull request, TeamCity ran the `test` task without any issue. It made me curious and I wanted to solve this issue.

{% highlight shell %}
$ ./gradlew test
> Task :app:testDebugUnitTest
...
java.lang.NoClassDefFoundError: jdk/internal/reflect/GeneratedSerializationConstructorAccessor1
        at jdk.internal.reflect.GeneratedSerializationConstructorAccessor1.newInstance(Unknown Source)
        at java.base/java.lang.reflect.Constructor.newInstanceWithCaller(Constructor.java:500)
        at java.base/java.lang.reflect.Constructor.newInstance(Constructor.java:481)
        at java.base/java.io.ObjectStreamClass.newInstance(ObjectStreamClass.java:1081)
        at java.base/java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2071)
        at java.base/java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1594)
        at java.base/java.io.ObjectInputStream.readObject(ObjectInputStream.java:430)
        at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:107)
        at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:64)
        at worker.org.gradle.process.internal.worker.GradleWorkerMain.run(GradleWorkerMain.java:62)
        at worker.org.gradle.process.internal.worker.GradleWorkerMain.main(GradleWorkerMain.java:67)
Caused by: java.lang.ClassNotFoundException: jdk.internal.reflect.GeneratedSerializationConstructorAccessor1
        at java.base/java.net.URLClassLoader.findClass(URLClassLoader.java:436)
        at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:588)
        at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:521)
        ... 11 more
...
{% endhighlight %}

> The error message could vary slightly depending on your _Jacoco_ tool version.

This error message showed some lines in `jdk`. Since there was no issue in TeamCity, it might be a bug in `jdk` and possibly I used a different `jdk` version from what TeamCity used.

{% include google-adsense-in-article.html %}

{% highlight shell %}
$ java -version
java version "12.0.2" 2019-07-16
Java(TM) SE Runtime Environment (build 12.0.2+10)
Java HotSpot(TM) 64-Bit Server VM (build 12.0.2+10, mixed mode, sharing)
{% endhighlight %}

TeamCity used Java 8 and my machine used Java 12. I updated my Java version to work on backend services, which now requires higher that Java 11. After updating, Gradle seemed work well. I could run the `assemble` task. The problem only happened for the `test` task strangely.

After researching deeply, this turned out a quite complex problem. In short, Java 8 and higher versions (e.g. Java 9, Java 10, ...) are different. Plugins or libraries should support higher versions to work properly. For me, they are _Jacoco_, _Mockito_, and _PowerMock_. There are quite typical plugins/libraries for Android testing.

For _Jacoco_, you can fix this issue by updating the plugin version and adding `jacoco.excludes`.([Gradle issue #5184](https://github.com/gradle/gradle/issues/5184)).

{% highlight groovy %}
jacoco {
    toolVersion = '0.8.4'
}

tasks.withType(Test) {
    jacoco.includeNoLocationClasses = true
    jacoco.excludes = ['jdk.internal.*']
}
{% endhighlight %}

_Mockito_ [had an update ([Mockito PR #1250](https://github.com/mockito/mockito/pull/1250)) to support this. If you see below error, you need to update `mockito-core` to use the latest version. The current project used `2.8.9` version which is quite old and didn't have that update. The latest _Mockito_ version at this point is `2.28.2` and `3.0.0`.

{% highlight groovy %}
$ ./gradlew test
> Task :app:testDebugUnitTest
...
com.project.ExampleTest > basicFunctionality FAILED
    org.mockito.exceptions.base.MockitoException at ExampleTest.java:36
        Caused by: java.lang.UnsupportedOperationException at ExampleTest.java:36
            Caused by: java.lang.IllegalArgumentException at ExampleTest.java:36
...
{% endhighlight %}

For me, there was another hurdle. It still didn't work and it was due to _PowerMock_. Unfortunately, it seemed not fixed yet ([Powermock issue #901](https://github.com/powermock/powermock/issues/901)).

{% highlight groovy %}
$ ./gradlew test
> Task :app:testDebugUnitTest
...
com.project.ExampleText > initializationError FAILED
    org.objenesis.ObjenesisException
        Caused by: java.lang.reflect.InvocationTargetException
            Caused by: java.lang.IllegalAccessError

...
{% endhighlight %}

If you don't use _PowerMock_ library, updating libraries and setting excludes of _Jacoco_ would fix your issue. In my case, however, it was tricky because of _PowerMock_. There might be some way to work around but it seemed not worth spending more time. As you know, the issue was caused by `jdk`. It means that this could be easily solved by using Java 8 instead of Java 12. `jenv`(https://www.jenv.be) helps to manage/switch multiple `jdk`s in the same machine. Using that, I was able to fix all my issue and to switch Java 12 when I need to work on the backend code.

{% highlight shell %}
$ jenv add /Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home
$ jenv versions
  oracle64-1.8.0.221
* oracle64-12.0.2 (set by /Users/sangsoo/.jenv/version)
$ jenv global oracle64-1.8.0.221
{% endhighlight %}

> If you use Android Studio to run gradle tasks including `test`, you might not see this issue. Android Studio allows you to set `jdk`. It could be different from your command line environment. By default, Android Studio uses an embedded JDK, which is Java 8.

![Android Studio Embedded JDK](/images/2019/08-15/android-studio-embedded-jdk.png)
