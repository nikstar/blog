---
title: "G Reader: RSS client for Telegram"
date: 2020-09-24T00:00:00+03:00
description: "G Reader is a new RSS bot for Telegram"
cover: "/img/greader-cover.png"
---

G Reader is a new RSS bot for Telegram. You can check it out here: [@GReaderBot](https://t.me/greaderbot). 

Every once in a while I come around an interesting blog I'd like to read in the future. If this blog is updated rarely (think once a year) it is easy to forget about it and never visit  again. These days Twitter is probably the most common way to follow interesting websites, but any updates there are likely to be missed.

Enter **RSS**, supported by every blog out there and still not dead despite Google's best efforts, and **G Reader**, my take on RSS client built as a Telegram bot.

<!-- ## Goals -->

From the start I knew I didn't want a separate app because I would never use it and the unread count would quickly become overwhelming. A bot naturally reappears at the top of messages list whenever a new item is posted like a normal channel.

I've tested half-dozen existing bots and all of them had the following subscribe flow:

```
/subscribe https://smittenkitchen.com/feed/
```

This is just not good enough. Firstly, you have to always use a command even though subscribing accounts for probably 95% of all actions. Secondly, most bots don't even try looking for a feed when given a link to a post instead of a direct feed URL.

Unsubscribing was also tedious and usually involved paging through the list of all subscriptions.

<!-- ## G Reader design -->

G Reader aims to solve these issues with simplified workflow.

Subscribing is as easy as sending a link. G Reader will do its beast to find the feed. You can always fall back to an RSS link, of course.

<!-- Video 1 here -->
<figure>
    <video controls muted height=600>
        <source type="video/webm" src="/img/greader-subscribe.webm">
        <source type="video/mp4"  src="/img/greader-subscribe.mp4">    
    </video>
    <figcaption>Subscribing in G Reader</figcaption>
</figure>

This enables subscribing by sharing an article directly from the browser. It feels very natural and is my favorite feature of G Reader.

<!-- Video 2 here -->
<figure>
    <video controls muted height=600>
        <source type="video/webm" src="/img/greader-share.webm">
        <source type="video/mp4"  src="/img/greader-share.mp4">    
    </video>
    <figcaption>Sharing from Chrome</figcaption>
</figure>

Unsubscribing was also simplified. It is done be replying `/unsubscribe` to any item in the feed you no longer want to read.

<!-- Video 3 here -->
<figure>
    <video controls muted height=600>
        <source type="video/webm" src="/img/greader-unsubscribe.webm">
        <source type="video/mp4"  src="/img/greader-unsubscribe.mp4">    
    </video>
    <figcaption>Unsubscribing</figcaption>
</figure>

I'm very happy with how G Reader turned out. It's live on Telegram as [@GReaderBot](https://t.me/greaderbot). If you have any feedback, feel free to [reach out](https://t.me/nikstar).

<!-- 
[//]: #  (why rss? 1) twitter easily missed 2) forget to visit directly )

why others bad? 1) message formatting 2) difficult to follow

why? 1) don't miss blogs that are rarely updated 2) easily follow 3) better than standalone app: will sometimes open 4) share sheet 5) supports groups

what it is not: 1) opinionated, no settings for formatting 2) intended for low volume

future steps: 1) localization 2) ompl support

-->
