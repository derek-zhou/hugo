---
title: Modal is considered harmful
slug: modal_is_considered_harmful.md
date: 2021-06-09
tags:
- web
description: Modal is a way to mimic a pop-up window in a webpage. Instead of a real native pop-up with all the annoying nature in it, a modal is implemented with HTML/CSS/Javascript, so it is actually just a part of the page, and only looks like a modal dialog. They are everywhere; but do they really make sense now? 
---

Unlike pop-ups, Modals are implemented with web technologies: HTML + CSS + Javascript. With a Modal, the original page is still intact but in the background, usually with some visual cue such as dropped shadow; and the user can only interact with a new and smaller portion of the page, the modal, which is cued to be in the forground. Just like a real dialog box, the user focus on the narrower task at hand, usually a form; and the user can safely go back to the previous page by either press a cancel button, a "close dialog" button, or simply click outside the modal. Modals are popularized by various frontend frameworks such as [Bootstrap](https://getbootstrap.com/docs/4.0/components/modal/) nearly 20 years ago and have been a mainstay in the web application arena ever since.

In this article, I will argue against them, from both a user experience perspective and a implementation perspective. There is no value in them anymore and web designers should take them behind the barn and shoot them.

## User experience hurdles ##

The modal draws parallel from a popup window in a native application; in reality, it is only a poor imitation. A main point of a real pop-up window is that the user can drag it, resize it, so the stuff blocked by the pop-up can still be visible. On the other hand, a html modal is neither dragable nor resizable; so it is just a fixed blocakge on the screen. The blocakge can even be significant on a phone screen. If your user still want to see any part of the background page, tough lock.

The modal makes poor use of the screen real estate. A modal usually have a faked title bar, extra bezel around, and submit/cancel buttons that take away precious screen space without adding functionality. This is especially bad on a small phone screen. On top of that, the click-out-side-the-modal-close-it behavior is very confusing; and the opposite, the do nothing behavior, is equally non-intuitive. There is simply no sane default for clicking outside a modal.

## Implementation caveats ##

The modal functionality is usually encapsulated in a client side javascript library so the invocation is easy ennough. However, it still changes the DOM, even if the change is small. If more than one party (for example, two diffentent clicne libraries, one client-side library and a server-side framework such as Phoenix Liveview) are modifying the DOM, some inconsistency can happen and can cause tricky corner cases.

The position and the size of a modal can also be tricky. If you are not careful, you could end up with a modal that is either too big, too small, or partially outside the viewport, etc, in a non conventional page layout or unusual client screen size.

## So what do we do? ##

In many cases, you don't need a modal. You can just squeeze the additional content in the page, after the user interaction triggering it. It is the same amount of DOM modification as a modal, however, the new content:

* is not blocking anything
* at the right place with the right context
* participates in the normal page layout flow

To reduce the clutter, you can have a control to remove/hide them either manually or automatically after it have serve its purpose.

## What if I really want the modal behavior ##

In some cases, you really want to limit the user's visibility and interaction to the new content. The solution is simple: Just make it a new screen. This is the age old solution and will not confuse anyone.

Now you may ask: how about performance? I don't want a blank screen or visible re-rendering when the user click the cancel button. Of course. Then we need to consider the technology in use:

* With modern client side frameworks on a semi-reasonable device, there will be no visual glitch even if you change the whole DOM. Assuming the data to render the old screen and the new screen are already local, any rendering is fast enough.
* With Phoenix LiveView, it can be a valid concern wanting to limit the DOM diff. However, there is a well known trick to just toggle the "hidden" attribute of the other DOM elements. So, you have a small difff and a fast screen transition.
* With plain old HTML, the cancel action can be implemented with `history.back();`. On modern browsers the re-rendering from the browser cache is instantaneous.

## Summary ##

HTML modal was a stop-gap invention, a compromise between user experience and what was possible 15 years ago. Now that we have better browsers, better devices and better software frameworks on both the client side and the server side, it is time to kiss html modals goodbye for good.
