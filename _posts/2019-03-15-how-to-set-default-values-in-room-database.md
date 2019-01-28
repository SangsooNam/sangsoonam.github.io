---
layout: post
title: "How to Set Default Values in Room Database"
---

When you initialize your application, you often need a default value to determine how it works. For example, most values of Settings require a default value. When you use a database, you read a value from there. Thus, it is better to put a default value in there instead of in the code.

How can you do it? Room database supports a callback method. There are two callbacks: `onCreate` and `onOpen`. `onCreate` is called only when the database is created for the first time. After that, this is not called again. `onOpen` is called when the database is opened. Both callbacks are not called when your application is started but called when your code accesses the database such as querying.

{% highlight java %}
public abstract static class Callback {

  public void onCreate(@NonNull SupportSQLiteDatabase db) {
  }

  public void onOpen(@NonNull SupportSQLiteDatabase db) {
  }
}
{% endhighlight %}

{% include google-adsense-in-article.html %}

If you want to pre-populate data only one time, you can do it using `onCreate` callback. However, it wouldn't fit well if there is a chance to add another default value later. If you think about Settings, there could be a new setting option in the next version.

When the application is restarted, `onOpen` can be called unlike `onCreate`. Now the remaining question is how to set default values properly. Since `onOpen` can be called several times, you need to pre-populate data only if it is necessary. If not, you could end up overriding data changed by a user.

This sounds like a bit tricky but actually quite simple to do. Room database supports to select a conflict algorithm. By selecting, `CONFLICT_IGNORE`, you can set default values without worrying overriding. Room database doesn't allow to access the database on the main thread unless you set `allowMainThreadQueries()`. Thus, the callback method is executed not on the main thread.

{% highlight java %}
final AppDatabase database = Room.databaseBuilder(this, AppDatabase.class, "database.db")
  .addCallback(new RoomDatabase.Callback() {

    @Override
    public void onOpen(@NonNull SupportSQLiteDatabase db) {
      super.onOpen(db);
      Map<String, String> defaultMaps = new HashMap<>();
      defaultMaps.put("theme", "dark");
      defaultMaps.put("auto_play_mode", "true");
      defaultMaps.put("shuffle_mode", "true");

      for (Map.Entry<String, String> entry : defaultMaps.entrySet()) {
        ContentValues values = new ContentValues();
        values.put("key", entry.getKey());
        values.put("value", entry.getValue());
        db.insert("Setting", SQLiteDatabase.CONFLICT_IGNORE, values);
      }
    }
  })
  .build();
{% endhighlight %}
