---
layout: post
title: "YAML vs JSON"
---

## JSON

JSON(JavaScript Object Notation) is wildly used for a client and a server communication. It is a simpler way to represent the data compared to XML(Extensible Markup Language). As you can see the below example, JSON is significantly less verbose than XML. Especially, there is no opening and closing tags in JSON.

<p class="code-label">JSON</p>
{% highlight json %}
{
  "person": {
    "firstname": "Tom",
    "lastname": "Smith",
    "year": 1982,
    "favorites": ["tennis", "golf"]
  }
}
{% endhighlight %}

<p class="code-label">XML</p>
{% highlight xml %}
<person>
  <firstname>Tom<firstname>
  <lastname>Smith<lastname>
  <year>1982<year>
  <favorites>
    <value>tennis</value>
    <value>golf</value>
  </favorites>
</person>
{% endhighlight %}

{% include google-adsense-in-article.html %}

## YAML
YAML(YAML Ain't Markup Language) gives a even simpler way than JSON to represent the data. It removes curly brackets(`{}`) and squard brackets(`[]`) except for inline collections. In addition, vertical alignment is used to show the structure. Technically, YAML is a superset of JSON. In most cases, it is possible to convert YAML to JSON and JSON to YAML. YAML also has extra features, which are not in JSON.

* **Commenting**: Adding additional information as a comment
* **Aliasing and Anchoring**: Giving an alias to an item and referring it instead of typing/repeating many times
* **Merging**: Merging aliased item into a current map

<p class="code-label">YAML</p>
{% highlight yaml %}
person:
  firstname: Tom
  lastname: Smith
  year: 1982
  favorites:
    - tennis
    - golf
{% endhighlight %}

### Basic Syntax
Here are basic syntaxes of YAML. I've added an equivalent JSON data so that you can compare between YAML and JSON.

#### Sequence
You can specify a sequence using `-`. You can nest a sequence with an empty `-`, followed by an indented sequence.

<p class="code-label">YAML</p>
{% highlight yaml %}
- foo
- bar
-
  - baz
  - qux
{% endhighlight %}

<p class="code-label">JSON</p>
{% highlight json %}
[
  "foo",
  "bar",
  [
    "baz",
    "qux"
  ]
]
{% endhighlight %}

#### Mapping
You can specify a key-value map using `:`. Each member of the map should be on a new line.

<p class="code-label">YAML</p>
{% highlight yaml %}
foo: value
bar:
  - baz
  - qux
{% endhighlight %}

<p class="code-label">JSON</p>
{% highlight json %}
{
  "foo": "value",
  "bar": [
    "baz",
    "qux"
  ]
}
{% endhighlight %}

#### Inline Sequence
You can make a sequence in a short way. Like JSON, `[]` is used for the inline sequence.

<p class="code-label">YAML</p>
{% highlight yaml %}
bar: [baz, qux]
{% endhighlight %}

<p class="code-label">JSON</p>
{% highlight json %}
{
  "bar": [
    "baz",
    "qux"
  ]
}
{% endhighlight %}

#### Inline Mapping
You can make a map like a JSON way with `{}`.

<p class="code-label">YAML</p>
{% highlight yaml %}
capitals: { South Korea: Seoul, Japan: Tokyo }
{% endhighlight %}

<p class="code-label">JSON</p>
{% highlight json %}
{
  "capitals": {
    "South Korea": "Seoul",
    "Japan": "Tokyo"
  }
}
{% endhighlight %}

#### Aliasing and Anchoring
You can give an alias to a specific item using `&` and name. Then, you can refer/anchor it with `*` and its name.

<p class="code-label">YAML</p>
{% highlight yaml %}
firstname: &firstname Tom
lastname: Smith
nickname: *firstname
{% endhighlight %}

<p class="code-label">JSON</p>
{% highlight json %}
{
  "firstname": "Tom",
  "lastname": "Smith",
  "nickname": "Tom"
}
{% endhighlight %}

Aliasing and anchoring is possible not only for a value but also for a whole item.

<p class="code-label">YAML</p>
{% highlight yaml %}
- person:
    firstname: Tom
    favorites: &toms_favorites [tennis, golf]
- person:
    firstname: John
    favorites: *toms_favorites
{% endhighlight %}

<p class="code-label">JSON</p>
{% highlight json %}
[
  {
    "person": {
      "firstname": "Tom",
      "favorites": [
        "tennis",
        "golf"
      ]
    }
  },
  {
    "person": {
      "firstname": "John",
      "favorites": [
        "tennis",
        "golf"
      ]
    }
  }
]
{% endhighlight %}

#### Merging
If you alias a map, you can merge that into the current map with `<<`.
<p class="code-label">YAML</p>
{% highlight yaml %}
- &twin_info  {
  lastname: Smith,
  year: 1982,
  favorites: [tennis, golf]
}
- person:
    firstname: Tom
    <<: *twin_info
- person:
    firstname: John
    <<: *twin_info
{% endhighlight %}

<p class="code-label">JSON</p>
{% highlight json %}
[
  {
    "lastname": "Smith",
    "year": 1982,
    "favorites": [
      "tennis",
      "golf"
    ]
  },
  {
    "person": {
      "firstname": "Tom",
      "lastname": "Smith",
      "year": 1982,
      "favorites": [
        "tennis",
        "golf"
      ]
    }
  },
  {
    "person": {
      "firstname": "John",
      "lastname": "Smith",
      "year": 1982,
      "favorites": [
        "tennis",
        "golf"
      ]
    }
  }
]
{% endhighlight %}

#### Multilines
You can put multilines just by adding it. In this case, one line break is ignored and consider it as one line. In this example, there is no `\n` between _serialization_ and _standard_.

<p class="code-label">YAML</p>
{% highlight yaml %}
YAML:
  YAML Ain't Markup Language

  YAML is a human friendly data serialization
  standard for all programming languages.
{% endhighlight %}

<p class="code-label">JSON</p>
{% highlight json %}
{
  "YAML": "YAML Ain't Markup Language\nYAML is a human friendly data serialization standard for all programming languages."
}
{% endhighlight %}

If you want to keep every line break, you can specify it using `|` with an indented block. This is treated as a literal block.

<p class="code-label">YAML</p>
{% highlight yaml %}
YAML: |
  YAML Ain't Markup Language

  YAML is a human friendly data serialization
  standard for all programming languages.
{% endhighlight %}

<p class="code-label">JSON</p>
{% highlight json %}
{
  "YAML": "YAML Ain't Markup Language\n\nYAML is a human friendly data serialization\nstandard for all programming languages."
}
{% endhighlight %}

#### Commenting
You can specify a comment using `#` to add an additional information.

<p class="code-label">YAML</p>
{% highlight yaml %}
# This is a comment
YAML: |
  YAML Ain't Markup Language
{% endhighlight %}

<p class="code-label">JSON</p>
{% highlight json %}
{
  "YAML": "YAML Ain't Markup Language\n\nYAML is a human friendly data serialization\nstandard for all programming languages."
}
{% endhighlight %}

## Conclusion
YAML has good extra features such as commenting, aliasing and anchoring. It is super human-readable. Compared to YAML, JSON is a bit limited. However, JSON is widely used and supported. One of the main reasons is that it is faster to serialize and deserialize, which is a key factor to communicate between a client and a server. Check [this benchmark](https://gist.github.com/havenwood/4513627). To sum up,

* Use JSON for communication between a client and a server.
* Use YAML for configuration files since YAML is really human-readable.

> If you want to try/learn YAML, this [YAML to JSON online](https://www.json2yaml.com/convert-yaml-to-json) site is quite handy.

## References
* [YAML official site](http://yaml.org/)
* [YAML vs JSON benchmark](https://gist.github.com/havenwood/4513627)
* [YAML to JSON online](https://www.json2yaml.com/convert-yaml-to-json)
