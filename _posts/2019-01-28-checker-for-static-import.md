---
layout: post
title: "Checker for Static Import"
---

When you write code, it is vital to keep consistency. It makes better readability and maintainability. It is easier when you work alone. However, it would be hard to keep it when many engineers work on the same project. Checkers are fit well for this situation. Among many consistency points, let's see how to keep consistency on a static import.

For example, Mockito has several methods where static import can be applied. That is widely used in the unit testing and static and non-static import can be mixed in the same file or in another file. This style inconsistency is what we want to avoid.

{% highlight java %}
@Before
public void setUp() {
    MockitoAnnotations.initMocks(this);
}

@Test
public void test() {
    ConnectionInfo connectionInfo = Mockito.mock(ConnectionInfo.class);
    ...
}
{% endhighlight %}

{% highlight java %}
import static org.mockito.Mockito.spy;

@Test
public void test() {
    AccountFragment accountFragment = spy(AccountFragment.class);
}
{% endhighlight %}

{% include google-adsense-in-article.html %}

[Checkstyle](http://checkstyle.sourceforge.net/) is a tool to check those style matters. There are lots of useful style rules but no rule for static import. Static import is highly related on the classes in your code or third-party libraries. So, you need to define a custom rule. First of all, you can set up the Checkstyle for Android project:

<p class="code-label">build.gradle</p>
{% highlight groovy %}
apply plugin: 'checkstyle'

checkstyle {
    toolVersion = '8.15'
}

task checkstyle(type: Checkstyle) {
    configFile rootProject.file("config/checkstyle/checkstyle.xml")
    source project.fileTree(dir: 'src', include: '**/*.java')
    classpath = files()
}

android {
  ...
}
{% endhighlight %}

Then, add a configure file with a custom rule. The rule uses a regular expression to find non-static Mockito imports. If there is a match, it is considered as an error.

<p class="code-label">config/checkstyle/checkstyle.xml</p>
{% highlight xml %}
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
    "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
    "https://checkstyle.org/dtds/configuration_1_3.dtd">

<module name = "Checker">
    <module name="TreeWalker">
        <module name="RegexpSinglelineJava">
            <property name="id" value="UseStaticImports" />
            <property name="format" value="(?&lt;!\.)\b(Mockito|MockitoAnnotations)\.[a-zA-Z]*\(" />
            <property name="message" value="Use static imports of Mockito for better readability." />
        </module>
    </module>
</module>
{% endhighlight %}


Now, you can run `checkstyle` task to see a result on your code. Below there are two static import errors at line 16 and 21.

{% highlight bash %}
$ ./gradlew checkstyle
Parallel execution with configuration on demand is an incubating feature.

> Task :app:checkstyle FAILED
[ant:checkstyle] [ERROR] ExampleUnitTest.java:16: Use static imports of Mockito for better readability. [UseStaticImports]
[ant:checkstyle] [ERROR] ExampleUnitTest.java:21: Use static imports of Mockito for better readability. [UseStaticImports]

FAILURE: Build failed with an exception.
...

{% endhighlight %}

{% highlight java linenos %}
package io.github.staticimportchecker;

import org.junit.Before;
import org.junit.Test;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.MockitoAnnotations;

public class ExampleUnitTest {

    @Mock
    private ConnectionInfo connectionInfo;

    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    public void test() {
        ConnectionInfo connectionInfo = Mockito.mock(ConnectionInfo.class);
    }
}

{% endhighlight %}

## Conclusion

Consistency is an important factor to make a solid code. Checkstyle is a handy checker in terms of styling during collaboration. After discussion, you can add a custom rule, such as static import, to keep the consistency now and future.

> If you want to try, you can clone [this repository](https://github.com/SangsooNam/static-import-checker) and run the task.
