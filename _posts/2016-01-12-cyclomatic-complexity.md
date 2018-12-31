---
layout: post
title: "Keep It Simple! Cyclomatic Complexity"
---

![Cyclomatic complexity](/images/2016/01-12/cyclomatic-complexity.png)

Whenever you write code, writing simple code is important. It will improve the maintainability. So, you and other developers can easily test, understand and modify. Here is an important question. How do you know that your code is simple? You may say that it is short so that it is simple. Short code is always good. However, we need a more objective view. For that, Cyclomatic complexity is a good measurement, used to indicate the complexity of code. It is computed using the control flow graph of the code.

If the code has lower cyclomatic complexity, there are lower risks to modify. The complexity **M** is defined as:

{% highlight sh %}
M = E - N + 2
  E = the number of edged of the graph
  N = the number of nodes of the graph
{% endhighlight %}

## Example 1

{% highlight javascript linenos %}
if (month == 1) {
  if (day == 1) {
    print('HAPPY NEW YEAR');
  } else {
    print('HAVE A NICE DAY');
  }
}
print('END');
{% endhighlight %}

From the above code, the control flow graph is depicted like below.

![Example Control Flow Graph ](/images/2016/01-12/example1.svg)

There are 7 nodes and 8 edges, so the complexity is 3 (8 - 7 + 2).

Theoretically, you need to think and draw the control flow graph to calculate the cyclomatic complexity. It could be really boring and be hard if the code is large. There is a much simpler way to calculate it.

{% highlight sh %}
M = B + 1
  B = the number of branch points(e.g. if, case, for, ||, while )
{% endhighlight %}

In this example, there are 2 branch points: `if(line 1)`, `if(line 2)`. Thus, the complexity is 3 (2 + 1), which is the same value.

{% include google-adsense-in-article.html %}

## Example 2
{% highlight javascript linenos %}
for (var i = 0; i < 10; i++) {
  if (i == 2 || i == 4 || i == 6) {
    print('2 OR 4 OR 6');
  }
}
print('END');
{% endhighlight %}

From the above code, the control flow graph is depicted like below.

![Example Control Flow Graph ](/images/2016/01-12/example2.svg)

There are 9 nodes and 12 edges, so the complexity is 5 (12 - 9 + 2). If you want to calculate it using a simpler way, count branch points. In this example, there are 4 branch points:`for(line 1)`, `if(line 2)`, `||(line 2)`, `||(line 2)`. So, the complexity is the same value, 5 (4 + 1).

> You can copy examples and check the complexity on [http://jscomplexity.org/](http://jscomplexity.org/)
