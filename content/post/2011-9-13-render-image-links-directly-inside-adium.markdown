---
layout: post
title: "Render image links directly inside Adium"
date: 2011-09-13T08:59:00Z
comments: false
permalink: /render-image-links-directly-inside-adium
tags:
---



Last night I delightfully discovered that Adium Message Styles are just html, css, and javascript rendered inside a webview. The next natural step was to write something in it, so I wrote a Message Style that tries to render any image link directly inline the conversation (campfire style).

![](/images/blog/adium1.png)

The code was written at midnight after a long day, so its not best. Basically, it's a setInterval that runs every 2.5 seconds that loops through all message elements, appending an img tag to the body of the message if an image link is detected. It also removes the processing class as to not reprocess the same messages.

Installation is simple, just download: 

[http://dl.dropbox.com/u/42561/Stockholm.AdiumMessageStyle.zip](http://dl.dropbox.com/u/42561/Stockholm.AdiumMessageStyle.zip)

and extract into ~/Library/Adium 2.0/Message Styles (create if necessary). Then choose the TOP Stockholm theme (no idea why there are two entries), and close your chat window. It should be activated next time a chat window opens.

![](/images/blog/adium2.png)
