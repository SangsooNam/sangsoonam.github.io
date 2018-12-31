---
layout: post
title: "AutoValue, Parcelable, and ParcelAdapter"
---

## AutoValue

AutoValue is a tool to eliminate cumbersome codes for value classes. For value classes, we commonly need to implement `equals`, `hashCode`, and `toString`. Those are really repetitive and easy to make hard-to-spot bugs if you miss to implement every method properly when you add a new field.

{% highlight java %}
public class Album {

    private String mTitle;
    private String mArtist;
    private String mDescription;
    private String mImage;

    public Album(String title, String artist, String description, String image) {
        mTitle = title;
        mArtist = artist;
        mDescription = description;
        mImage = image;
    }

    public String getTitle() {
        return mTitle;
    }

    public String getArtist() {
        return mArtist;
    }

    public String getDescription() {
        return mDescription;
    }

    public String getImage() {
        return mImage;
    }
}
{% endhighlight %}

AutoValue provides an easier way to create immutable value classes using an `@AutoValue` annotation. To use AutoValue, there are two simple steps. First, add `@AutoValue` annotation into your value class. Then, make your class and fields as abstract.

{% highlight java %}
@AutoValue
public abstract class Album {

    @NonNull public abstract String title();
    @NonNull public abstract String artist();
    @Nullable public abstract String description();
    @NonNull public abstract String image();

    public static Album create(String title, String artist, String description, String image) {
        return new AutoValue_Album(title, artist, description, image);
    }
}
{% endhighlight %}

> AutoValue generates a subclass while compiling. If you have _"Cannot resolve symbol ..."_ warning, build your project. Then, it will be resolved properly.

After building your project, you can see the generated subclass, `AutoValue_Album`. It implements `equals`, `hashCode`, and `toString`. Also, it checks the nullability according to `@NonNull` and `@Nullable` annotation.

{% highlight java %}
final class AutoValue_Album extends Album {

  private final String title;
  private final String artist;
  private final String description;
  private final String image;

  AutoValue_Album(
      String title,
      String artist,
      @Nullable String description,
      String image) {
    if (title == null) {
      throw new NullPointerException("Null title");
    }
    this.title = title;
    if (artist == null) {
      throw new NullPointerException("Null artist");
    }
    this.artist = artist;
    this.description = description; // No null check since nullable
    if (image == null) {
      throw new NullPointerException("Null image");
    }
    this.image = image;
  }

  @NonNull
  @Override
  public String title() {
    return title;
  }

  @NonNull
  @Override
  public String artist() {
    return artist;
  }

  @Nullable
  @Override
  public String description() {
    return description;
  }

  @NonNull
  @Override
  public String image() {
    return image;
  }

  @Override
  public String toString() {
    return "Album{"
        + "title=" + title + ", "
        + "artist=" + artist + ", "
        + "description=" + description + ", "
        + "image=" + image
        + "}";
  }

  @Override
  public boolean equals(Object o) {
    if (o == this) {
      return true;
    }
    if (o instanceof Album) {
      Album that = (Album) o;
      return (this.title.equals(that.title()))
           && (this.artist.equals(that.artist()))
           && ((this.description == null) ? (that.description() == null) : this.description.equals(that.description()))
           && (this.image.equals(that.image()));
    }
    return false;
  }

  @Override
  public int hashCode() {
    int h = 1;
    h *= 1000003;
    h ^= this.title.hashCode();
    h *= 1000003;
    h ^= this.artist.hashCode();
    h *= 1000003;
    h ^= (description == null) ? 0 : this.description.hashCode();
    h *= 1000003;
    h ^= this.image.hashCode();
    return h;
  }
}
{% endhighlight %}

{% include google-adsense-in-article.html %}

## Parcelable

`Parcelable` was introduced in Android to store values mush fast than `Serializable`. It is important to make it parcelable as mush as possible because mobile devices can gain performance improvement a lot from that. To make value classes as parcelable, there are some requirements.

* Classes must implement `Parcelable` interface.
* Classes implementing the `Parcelable` interface must also have a non-null static field called `CREATOR` of a type that implements the `Parcelable.Creator` interface.

Even though there is only one field variable, you need to write many lines to implement `Parcelable` like this to fit requirements.

{% highlight java %}
public class MyParcelable implements Parcelable {
    private int mData;

    public int describeContents() {
        return 0;
    }

    public void writeToParcel(Parcel out, int flags) {
        out.writeInt(mData);
    }

    public static final Parcelable.Creator<MyParcelable> CREATOR
            = new Parcelable.Creator<MyParcelable>() {
        public MyParcelable createFromParcel(Parcel in) {
            return new MyParcelable(in);
        }

        public MyParcelable[] newArray(int size) {
            return new MyParcelable[size];
        }
    };

    private MyParcelable(Parcel in) {
        mData = in.readInt();
    }
}
{% endhighlight %}

Whenever you want to make parcelable value classes, you need to implement those. Like `equals` and other methods, those are also boilerplate codes. By default, AutoValue doesn't support `Parcelable` because it was implemented mainly for Java not for Android. However, it supports to make extensions. One extention, called [_auto-value-parcel_](https://github.com/rharter/auto-value-parcel), support it so we can remove those code. To use that extension, you first need to add below dependency into your `build.gradle`.

{% highlight groovy %}
dependencies {
  compile 'com.ryanharter.auto.value:auto-value-parcel:0.2.5'
}
{% endhighlight %}

Then, just add `implments Parcelable` in your class. That' it! You don't have to implement `CREATOR`, `writeToParcel` and etc. Parcel extension generates those for you.

{% highlight java %}
@AutoValue
public abstract class Album implements Parcelable {

    @NonNull public abstract String title();
    @NonNull public abstract String artist();
    @Nullable public abstract String description();
    @NonNull public abstract String image();

    public static Album create(String title, String artist, String description, String image) {
        return new AutoValue_Album(title, artist, description, image);
    }
}
{% endhighlight %}

Here is the generated class from parcel extension.
{% highlight java %}
final class AutoValue_Album extends $AutoValue_Album {
  public static final Parcelable.Creator<AutoValue_Album> CREATOR = new Parcelable.Creator<AutoValue_Album>() {
    @Override
    public AutoValue_Album createFromParcel(Parcel in) {
      return new AutoValue_Album(
          in.readString(),
          in.readString(),
          in.readInt() == 0 ? in.readString() : null,
          in.readString()
      );
    }
    @Override
    public AutoValue_Album[] newArray(int size) {
      return new AutoValue_Album[size];
    }
  };

  AutoValue_Album(String title, String artist, String description, String image) {
    super(title, artist, description, image);
  }

  @Override
  public void writeToParcel(Parcel dest, int flags) {
    dest.writeString(title());
    dest.writeString(artist());
    if (description() == null) {
      dest.writeInt(1);
    } else {
      dest.writeInt(0);
      dest.writeString(description());
    }
    dest.writeString(image());
  }

  @Override
  public int describeContents() {
    return 0;
  }
{% endhighlight %}

## ParcelAdapter (Optional)

Parcel extension support all of the types supported by [Parcel](https://developer.android.com/reference/android/os/Parcel.html) class. However, there is a bit performance problem since it uses generic methods for `List` or custom classes. Let's say you now have `List` data in your class.

{% highlight java %}
@AutoValue
public abstract class Album implements Parcelable {

    @NonNull public abstract String title();
    @NonNull public abstract List<Artist> artists();
    @Nullable public abstract String description();
    @NonNull public abstract String image();

    public static Album create(String title, List<Artist> artists, String description, String image) {
        return new AutoValue_Album(title, artists, description, image);
    }
}
{% endhighlight %}

Parcel extension generates `writeToParcel` method like this.
{% highlight java %}
@Override
public void writeToParcel(Parcel dest, int flags) {
  dest.writeString(title());
  dest.writeList(artists());  // This is a generic method.
  if (description() == null) {
    dest.writeInt(1);
  } else {
    dest.writeInt(0);
    dest.writeString(description());
  }
  dest.writeString(image());
}
{% endhighlight %}

The performance problem happens from the `writeList` method. Value classes implementing `Parcelable` interface have a non-null static field `CREATOR`. That field has a logic to construct an object again from `Parcel`. Generic methods, such as `writeList` and `writeParcelable`, write a full value class path to access `CREATOR` field later. For example, it uses 64 bytes for the `Album` class if the full class path is _sangsoonam.github.io.model.Album_. Each character takes 2 bytes and it has 32 characters.

There is another method called `writeTypedList`. This method writes doesn't wrtie the full class path. To read the data written with `writeTypedList`, you need to use a `readTypedList` method. `readTypedList` method has a `Creator` parameter so the value class knows how to construct an object without knowing a full class path. In short, `wrtieList` writes some info to access `CREATOR` later but `writeTypedList` doesn't since it is set in the code.

> For custom classes, check a `writeTypedObject` method.

You may think that is a small difference, but it is actually quite big. For the same data, it could be 344 bytes vs 44 bytes. If you want to see, check [this repository](https://github.com/SangsooNam/autovalue-parceladapter) and run unit tests.

To use non generic method, you need to define a custom de/serialization logic. `TypeAdapter` allows you to do but it is optional because it requries a small runtime component. To use that, you need to add this dependency.

{% highlight groovy %}
dependencies {
  compile 'com.ryanharter.auto.value:auto-value-parcel-adapter:0.2.5'
}
{% endhighlight %}

Then, create a `ListTypeAdapter` implementing `TypeAdapter` in the `Artist` class.

{% highlight java %}
@AutoValue
public abstract class Artist implements Parcelable {

    @NonNull public abstract String name();

    public static Artist create(String name) {
        return new AutoValue_Artist(name);
    }

    public static final class ListTypeAdapter implements TypeAdapter<List<Artist>> {
        @Override
        public List<Artist> fromParcel(Parcel in) {
            return in.createTypedArrayList(new Creator<Artist>() {
                @Override
                public Artist createFromParcel(Parcel source) {
                    return AutoValue_Artist.CREATOR.createFromParcel(source);
                }

                @Override
                public Artist[] newArray(int size) {
                    return new Artist[size];
                }
            });
        }

        @Override
        public void toParcel(List<Artist> value, Parcel dest) {
            dest.writeTypedList(value);
        }
    }
}
{% endhighlight %}

Finally, add `@ParcelAdapter` annotation with `ListTypeAdapter` class to define a custom de/serialization.

{% highlight java %}
@AutoValue
public abstract class Album implements Parcelable {

    @NonNull public abstract String title();
    @ParcelAdapter(Artist.ListTypeAdapter.class)
    @NonNull public abstract List<Artist> artists();
    @Nullable public abstract String description();
    @NonNull public abstract String image();

    public static Album create(String title, List<Artist> artists, String description, String image) {
        return new AutoValue_Album(title, artists, description, image);
    }
}
{% endhighlight %}



## References

* [AutoValue (https://github.com/google/auto/blob/master/value/userguide/index.md)](https://github.com/google/auto/blob/master/value/userguide/index.md)
* [AutoValue Parcel (https://github.com/rharter/auto-value-parcel)](https://github.com/rharter/auto-value-parcel)
* [AutoValue ParcelAdapter (https://github.com/rharter/auto-value-parcel#typeadapters)](https://github.com/rharter/auto-value-parcel#typeadapters)
* [Example Repository (https://github.com/SangsooNam/autovalue-parceladapter)](https://github.com/SangsooNam/autovalue-parceladapter)
