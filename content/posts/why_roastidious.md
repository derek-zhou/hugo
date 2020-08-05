---
title: Why Roastidio.us?
slug: why_roastidious
date: 2020-07-25
tags:
- roastidious
description: Roastidious is a open platform for people to post comments about any webpage.
---

[Roastidio.us](https://roastidio.us) is a open platform for people to post comments about any webpage. It is, first and foremost, a blogger's tool. I feel that it is fitting to explain the ideas behind roastidio.us in a blog post.

## Static blog ##

Like many blogs out there, this blog is static. More and more bloggers are migrating to a static platform, mostly due to the following reasons:

 * Security
 * Avoiding obsolescence
 * Cost and Performance
 * Avoiding lock-in

There are many excellent static blog generators out there. To name a few:

 * [Jekyll](https://jekyllrb.com/)
 * [Hugo](https://gohugo.io/)
 * [Pelican](https://blog.getpelican.com/)
 * [Nikola](https://getnikola.com/), which powers this blog

However, one feature that bloggers missed in a static blog is an matching commenting system. Yes, you could use social media, but that feels like caving into the evil empires. Or you can use `Disqus!`, but that also has privacy implication, and you have to put up with unknown third-party javascript blob on your site. 

## What Roastidio.us provides ##

Roastidio.us is the perfect complement to static blogs. It solved the problem of soliciting, censoring, and replying to comments in an elegant way.

### Simple to setup ###

As the blogger, you only need to do a straight forward setup task ahead of time to claim your blog space. First, you tell Roastidio.us where your webspace is. Webspace is a directory in a URL: For example, "https://google.com/path/" is a webspace, which covers "https://google.com/path/a.html" and "https://google.com/path/b.html," but not "https://google.com/path/c/d.html." Therefore, each web page in the universe belongs to one and only one webspace. Then Roastidio.us will challenge you with a simple task to install a unique meta line in your index page. If you can complete the trick, you own your blog. The process is simple, universally effective, and secure. I took the inspiration from [certbot](https://certbot.eff.org/), which I, among millions of others, depend on for our SSL certificates.

### Simple to invoke ###

You only need to link to [https://roastidio.us/roast](https://roastidio.us/roast) in your blog post to direct your visitors to the correct place to comment on your blog page. Roastidio.us uses the [referer header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer) to figure out where the request to roast is coming from; that's all it needs. There is no javascript blob, HTML snippet, image. It is a plain link, and you decide where to put it and how you like to word it. It is that simple.

### Email notifications ###

If someone comments on your blog post, you would receive an email from Roastidio.us, including the full text of the comment. You and you alone decide whether the message is worthy of being shown to the general public. You can also reply to comments on Roastidio.us, and the other party would receive an email with full text as well. The emails are correctly threaded, have the proper subject, have no tracking pixels or any external reference, so they can also serve as long term archiving means.

### Fair and autonomous rules ###

Roastidio.us has unique rules that are fair and autonomous. I think the existence of SPAMs is a fact of life; as long as they are not snowballing, the best way to deal with them is to ignore them. Within the rules of roastidio.us, no one can SPAM you repeatedly, or SPAM a broader audience without your approval. Even if you accidentally let some SPAM through, they will not last very long. 

### Proper threading ###

Most forum software now days is flat threaded, i.e., appending all comments to the end. However, replies in roastidio.us are organized and presented as a tree, like good old email threads, even on narrow phone screens. There are some tricks to make this presentation both practical and nice looking. Try it and give me feedback!

## Beyond blogging ##

Roastidio.us is not limited to static blogs. You can use roastidio.us anywhere, either with a CMS based blog like WordPress or Joomla!, or some site that is not a blog at all. One can comment on any web pages, claimed or unclaimed, as long as it has a URL accessible to the general public. The same rules apply, so feel free to comment on anything while protecting your privacy and freedom of speech. 

> Honor, privacy, and freedom of speech; Defend them or lose them. -- Derek Zhou

