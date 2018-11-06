---
layout: post
title: "Alfred Extension for creating Wunderlist tasks"
date: 2012-01-09T23:36:00Z
comments: false
permalink: /alfred-extension-for-creating-wunderlist-task
categories:
---



While looking for a way to add wunderlist tasks via alfred, I came upon [wunderlist-for-alfred](http://jdfwarrior.tumblr.com/post/13163220116/wunderlist-for-alfred)

Looked cool, but I wanted to write my own that didn't depend on php.

I used ```lsof``` to figure out the location of the db, then used ```file``` to see what kind of db it was. Luckily, it was sqlite3, so I was able to poke around and figure out the sql to create a task.

Here's the alfred extenstion that ties it all together:

```sh
user=`whoami`
wunderdb="/Users/$user/Library/Wunderlist/wunderlist.db"
sqlite3 $wunderdb "insert into tasks (name, list_id) values ('{query}', 1)"
```

Download it [here](http://dl.dropbox.com/u/42561/wunderlist-capotej.alfredextension)
