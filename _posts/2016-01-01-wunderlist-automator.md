---
layout: post
title: "Add Task to Wunderlist with Automator Service"
---

![](/images/2016/01-01/wunderlist-automator.png)

[Wunderlist](https://www.wunderlist.com/) is a handy task management application which supports a synchronization across all major devices. These days, I'm using it not only for my tasks but also for memos about ideas and references. I usually add a task inside the application. However, I often just copy a text and add a task by pasting. You might notice that it would be a quite tedious task.

Here is how I simplify it.

## Wunderlist Service

If you are using Wunderlist3, Wunderlist already added a "Add to Wunderlist" service in your mac. To use that, just select "Add to Wunderlist" in the context menu after selecting a text.

![Wunderlist Service](/images/2016/01-01/wunderlist-service.png)

By registering a keyboard shortcut, it is much simpler.

![Wunderlist Keyboard Shortcut](/images/2016/01-01/wunderlist-service-keyboard.png)

{% include google-adsense-in-article.html %}

## Automator Service

Default Wunderlist service looks great until I notice two things.

- New task always goes to _'Inbox'_ list. Even though I select another list, new task doesn't go to the selected list.
- Selected text is used for both new task's title and note. It looks weird to have the duplicate text. I prefer to put the selected text only on the title.

I've fixed it with a simple automator service with AppleScript. After following steps, you can use it in the similar way, select "Add to Wunderlist3" in the context menu after selecting a text.

![Automator Service](/images/2016/01-01/automator-service.png)

### Simple Way to Setup

1. Download [_'Add to Wunderlist 3'_](/images/2016/01-01/Add to Wunderlist 3.tgz) and Unzip
2. Execute _'Add to Wunderlist 3.workflow'_ and install it.

### Manual Way to Setup

![Automator Service Setup](/images/2016/01-01/automator-service-setup.png)


1. Launch _'Automator'_
2. Choose _'Service'_ in the _'Choose a type for your document:'_ popup
3. Add an item _'Run AppleScript'_ from the library
4. Copy and paste below apple script code
5. Save as _'Add to Wunderlist 3'_

{% gist SangsooNam/588aff22906c698374d5 %}

> To hide the default Wunderlist service, go to the keyboard shortcuts and unselect it.
![Wunderlist Service Hide](/images/2016/01-01/wunderlist-service-hide.png)
