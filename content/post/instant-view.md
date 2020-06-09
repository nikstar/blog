+++
date = "2020-06-05T16:42:00+03:00"
title = "Instant View for Custom Domains"
description = "Steps to add Telegram Instant View support for custom domains"
featured = false
+++

If you want to add Instant View support for your website, the only official recourse you have is to [create a template](https://instantview.telegram.org/my/), submit it, [request](https://instantview.telegram.org/contest#target-domains) for it to be included and [wait](/img/soon.webp). There is no official word for how long it takes to review or even a guarantee that your domain will ever be accepted. I haven't heard of any cases where templates for small blogs or media were added after following this process.

Luckily there's another way to get Instant View by disguising your page as a Medium article. This includes adding some meta tags and slightly modifying page structure. 

Big thank you to folks in [Russian Instant View chat](https://t.me/instantview_russian) for discovering this. 

## Meta tags

The following two tags will trick Telegram into using Medium template for our site.

+ ```html
  <meta property="al:android:app_name" content="Medium" />
  ```
  This will make Telegram use Medium template for our article. As far as we know this does not break anything, even if user has Android Medium app installed.

+ ```html
  <meta property="article:published_time" content="2020-02-02T00:00:00.000Z" />
  ```
  Setting date is required due to specifics of Medium template.

Additional tags are used to set up _webpage preview_ (the block you see in the chat above Instant View button). These are optional.

+ ```html
  <meta property="og:site_name" content="SITE_NAME" />
  ```
  Shows correct site name instead of default "Medium".

+ ```html
  <meta property="og:description" content="DESCRIPTION"  />
  ```
  A short description.

+ ```html
  <meta property="og:image" content="PREVIEW_IMAGE_URL" />
  ```
  A preview image.

Following tags modify parts of the Instant View page itself.

+ ```html
  <meta name="author" content="AUTHOR_NAME" />
  ```
  Author name will be shown below title next to the date. Add multiple tags for multiple authors.

+ ```html
  <meta name="telegram:channel" content="@YOUR_CHANNEL />
  ```
  A link to a Telegram channel with a "Join" button can be shown at the top of the article. Note that this value must start with `@`.


## Page structure

*You can refer to the source of this article as a starting point.*

The best way to make your page work is to use <i><a href="https://instantview.telegram.org/docs#supported-types">native IV markup</a></i>, i.e. something that would work with the most basic template:

```xpath
~version: "2.1"
body: //article
title: //article//h1
```

Start by placing article content inside `article` tag and making sure title uses `h1` tag. This should be enough to make Instant View work. From this point continue to adapt the source code fixing any issues by changing tags to what Telegram expects: refer to the "HTML counterpart" column in [this table](https://instantview.telegram.org/docs#supported-types). Use [@WebpageBot](https://t.me/WebpageBot) to force Instant View to update.

Place elements you wish to remove in Instant View inside `header` and `footer` tags. For example, on this page date is placed in `article header div` and is not shown in Instant View. Title (`article header h1`) is set before the entire header is removed.

Cover image (shown above title inside Instant View) can be added be placing following block inside `article`.

```html
<section class="is-imageBackgrounded">
  <figure>
    <img src="https://i.imgur.com/ea3mHTv.png"/>
  </figure>
</section>
```

This code is specific to Medium template.

In fact, instead of using `article` and other semantic tags as suggested above, you can go ahead and match page structure of any Medium article, and this should generate Instant View just as well. Copy [sample Medium template](https://instantview.telegram.org/samples/medium.com) into [a custom template](https://instantview.telegram.org/my) to debug any issues.


## Conclusion

Unfortunately, not all available [properties](https://instantview.telegram.org/docs#instant-view-format) can be set while using Medium template. Looks like there is no way to set `kicker`, `author_url` and `document_url`. If you find a way, please share it [with me][me].

And this is it. This method is a good solution for standalone blogs and small-scale websites that are unlikely to receive official support soon.

If you believe significantly changing page structure would be too difficult or bad for SEO, you might want to create a special version of each page similar to Google's AMP and then redirect Telegram crawler to it. Here is a snippet for nginx:

```perl
if ( $http_user_agent ~ 'TelegramBot' ) {
    return 301 /iv/$request_uri;
}
```

In this example `/iv/hello.html` is a simplified, Instant View compatible version of `/hello.html`. This was the [first method](/blog/instant-view-for-custom-domain) discovered before the Medium trick was found.

Thanks for reading. Feel free to [reach out][me] if you have any questions.

[me]: https://t.me/nikstar