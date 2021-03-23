---
title: To Use Roastidio.us
slug: use_roastidious
date: 2020-07-25
tags:
- roastidious
description: Roastidious needs an URL to tell it what the topic is.
---

[Roastidio.us](https://roastidio.us) only needs an URL to tell it what the topic is. Usually, you don't want to type in URLs, as they can be very long. There are several ways to input URLs; not all of them are obvious.

## Use the link ##

The easiest way to invoke Roastidio.us is to click the link embedded on the web page. Roastidio.us uses the [referer header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer) to figure out where the request to roast is coming from; that's all it needs. The link at the bottom of this blog post is a good example. 

One thing to keep in mind is you have to make sure the referer header is intact. In Firefox 87+ the default of referrer policy is very strict, it will strip off path and query string when crossing origins. So you would need to add the proper `referrerpolicy` attribute in the link:

``` html
<a href="https://roastidio.us/roast" referrerpolicy="no-referrer-when-downgrade">Roast me at Roastidio.us!</a>

```

The downside is the web page needs to provide the link. If you want to roast a web page that does not offer such a hyperlink, you'll have to do something else.

## Paste the URL ##

You can also copy the address from the location bar from your browser, open a tab to roastidio.us then paste the URL. To speed up the process, you may want to bookmark roastidio.us, or even has it in a pinned tab, so it is readily available.

It is undoubtedly more work than the first method. Is there a way that requires less clicking? Yes.

## Use the bookmarklet ##

[Bookmarklet](https://en.wikipedia.org/wiki/Bookmarklet) is a fairly old technology that recently fell out of favor. The idea is to bookmark not Roastidio.us, but a simple javascript snippet that looks into the current location bar and feed that info into Roastidio.us. It looks like this:

```javascript
javascript:location.href='https://roastidio.us/roast?url='+encodeURIComponent(window.location.href)
```

If you have some basic understanding of javascript, you can easily see what it is trying to do. On a desktop browser, you can right-click the above link and bookmark it and give it a name you can easily remember (such as: Roast it on roastidio.us). Next time you want to roast something, you can invoke this bookmarklet from either your bookmark bar or bookmark menu. Simple. All major versions of browsers still support bookmarklets.

However, a mobile browser does not allow you to right-click or bookmark a link. You can still bookmark a page, but the link above is not a page, but a bridge into Roastidio.us, so what do you do? To add a bookmarklet, you will need to:

 1. bookmark any page, such as this page
 1. select the above link and copy to clipboard
 1. long touch the bookmark you created, choose edit
 1. paste the link you just copied into the location box, and change the name of the bookmark

The way to invoke bookmarklet is also different. Most mobile browsers will open a new tab before going to the bookmark you choose from the menu, and in our case, that defeats the purpose of the bookmarklet, since the javascript snippet would get a useless blank location. The correct way is different depends on what browser you are using:

 * On Chrome mobile: Touch the address bar, type the first few letters of the name of this bookmarklet, then choose it from the drop-down menu
 * On Firefox mobile: Touch the address bar, touch the bookmark tab, then choose this bookmarklet from the list

So it is only slightly more complicated than doing the same on a desktop browser. There is a more detailed write up on [this site:](https://www.zotero.org/download/bookmarklet). The down side is each browser is slightly different; also [CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) may impede user's ability to use bookmarklet on a website.

## Even more natural ways ##

It is not hard to turn the bookmarklet idea into a browser extension, or (heaven forbid) a mobile app so that one can click less. I don't feel like doing it now.
