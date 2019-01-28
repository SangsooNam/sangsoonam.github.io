---
layout: post
title: "App Crash by `@NonNull` and Android Room"
---

## @NonNull

`@NonNull` is a handy annotation to help avoid passing a wrong value in fields and parameters. When Android Studio detects, it highlights those potential problem points.

![NonNull](/images/2018/12-12/set-item.png)

This annotation is totally optional. However, I believe that having it is one of the good practices and bringing much bigger value in terms of a bug prevention. This annotation this won't be retained by the VM at runtime so you don't have to worry about the performance of your application.

{% highlight java %}
@Documented
@Retention(CLASS) // Not retained by the VM at runtime.
@Target({METHOD, PARAMETER, FIELD, LOCAL_VARIABLE, ANNOTATION_TYPE, PACKAGE})
public @interface NonNull {
}
{% endhighlight %}

If you care about the code quality, I'm sure you would love this annotation and want to apply as many places as possible. It will make your code more solid, especially on `NullPointerException`. This annotation has been working great so far.  **Recently, however, I've found that it can cause a crash when you use Android Room.**

{% include google-adsense-in-article.html %}

## Android Room

The Android Room is a persistence library that provides an abstraction layer over SQLite. It allows us more robust database access while supporting the full features of SQLite. Entity is one of three major components in Room and it represents a table within the database. This is a entity example.

{% highlight java %}
@Entity
public class User {
    @PrimaryKey(autoGenerate = true)
    public int uid;

    public String email;

    public String name;
}
{% endhighlight %}

This entity has three fields. Unless you set to ignore, each field will represent each column in a table. Suppose you released the app with this entity. In the application, an email value is considered as not null. Since you now know `@NonNull` annotation, you could eager to apply it. So far, that has been recommended without any doubt. However, you should be cautious for `@Entity` class because that will make a crash.

`@NonNull` has been just an additional information during the development. However, there is a side effect when it is used in `@Entity` class. Android Room is based on SQLite and Entity represents a table. In the table, a field has a `notNull` attribute, which prevents from putting a null value in the table. You can check the table schema by setting `room.schemaLocation`.

<p class="code-label">build.gradle</p>
{% highlight groovy %}
android {
  ...
  defaultConfig {
    javaCompileOptions {
      annotationProcessorOptions {
        arguments = ["room.schemaLocation": "$projectDir/schemas".toString()]
      }
    }
  }
}
{% endhighlight %}

{% highlight bash %}
$ cat app/schemas/io.github.sangsoonam.room_nonnull.db.AppDatabase/1.json
{
  "formatVersion": 1,
  "database": {
    "version": 1,
    "identityHash": "...",
    "entities": [
      {
        "tableName": "User",
        "createSql": "CREATE TABLE IF NOT EXISTS `${TABLE_NAME}` (`uid` INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, `email` TEXT, `name` TEXT)",
        "fields": [
          {
            "fieldPath": "uid",
            "columnName": "uid",
            "affinity": "INTEGER",
            "notNull": true
          },
          {
            "fieldPath": "email",
            "columnName": "email",
            "affinity": "TEXT",
            "notNull": false #<== `@NonNull` will change this value
          },

{% endhighlight %}

When you set `@NonNull` to a file in `@Entity` class, Andorid Room considers you want to make a not null field in the table. Since the database schema is changed, you need to handle the migration. Otherwise, there will be a crash with this exception.

{% highlight java %}
@Entity
public class User {
    @PrimaryKey(autoGenerate = true)
    public int uid;

    @NonNull
    public String email;

    public String name;
}
{% endhighlight %}

{% highlight java %}
java.lang.IllegalStateException: Room cannot verify the data integrity. Looks like you've changed schema but forgot to update the version number. You can simply fix this by increasing the version number.
  at android.arch.persistence.room.RoomOpenHelper.checkIdentity(RoomOpenHelper.java:135)
  at android.arch.persistence.room.RoomOpenHelper.onOpen(RoomOpenHelper.java:115)at android.arch.persistence.db.framework.FrameworkSQLiteOpenHelper$OpenHelper.onOpen(FrameworkSQLiteOpenHelper.java:151)
{% endhighlight %}

So, `@NonNull` requires you a migration logic. It seems simple. You just need to change the column attribute along with updating the database version. When you try it, you will realize that it is supposed to be simple but almost impossible. **SQLite is a foundation of Android Room. It doesn't support altering column.**

> *Only the RENAME TABLE, ADD COLUMN, and RENAME COLUMN variants of the ALTER TABLE command are supported. Other kinds of ALTER TABLE operations such as DROP COLUMN, ALTER COLUMN, ADD CONSTRAINT, and so forth are omitted.*
>
> *[SQL Features That SQLite Does Not Implement
](https://www.sqlite.org/omitted.html)*

## Conclusion

Because of that, I recommend not to use `@NonNull` annotation to fields in `@Entity` class. However, it doesn't mean you cannot use that annotation at all in `@Entity` class. **Except for fields, you can put it to constructor parameters or getters without any side effect.**

{% highlight java %}
@Entity
public class User {
    @PrimaryKey(autoGenerate = true)
    private int uid;

    // WARNING: No `@NonNull` annotation
    private String email;

    private String name;

    public void setEmail(@NonNull String email) {
        this.email = email;
    }

    @NonNull
    public String getEmail() {
        return email;
    }
    ...
}
{% endhighlight %}
