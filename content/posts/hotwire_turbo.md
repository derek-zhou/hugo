---
title: Hotwire Turbo Experience
slug: hotwire_turbo
date: 2021-01-15
tags:
- web
description: Turbo is a new way to communicate between the javascript client side and the serverside, developed by the great folks at Basecamp. This blog post is a summary of my brief experience with Turbo
---

On Christmas 2020, Basecamp released the brand new web application framework [Hotwire](https://hotwire.dev/) that powers their new [hey.com](https://hey.com) web mail platform. It contains 3 parts:

* Turbo, which is a client library that handle communication with the server
* Stimulus, which is a javascript framework for reusable components
* Strada, which is a way to unify web client and native mobile client

Strada is not yet released. Stimulus has been there for a while. In this blog post, I focus on Turbo and share my experiences. I've always admired the work done be Basecamp, and [Roastidio.us](https://roastidio.us) is using [Trix](https://trix-editor.org/), another Basecamp contributed javascript library since day one. Let's find out more about Turbo!

## Turbo's components ##

Turbo actually has 3 loosely coupled components that can be used independently:

* Turbo Drive
* Turbo Frames
* Turbo Stream

Let's look at them one by one.

## Turbo Drive ##

Turbo Drive is not really new, but is a modern rewrite of the venerable [Turbolink](https://github.com/turbolinks/turbolinks), also from Basecamp. Turbo drive is a "magic" javascript library that turn plain links in your webpage into pure javascript interactions to load the webpages in the background, to greatly reduce the glitchy experience of a regular link. And it very easy to use; you added one library, changed a couple places in your javescript and it is done. There is no change needed on the server side; It works everywhere. This rewrite replaced the old XMLHttpRequest based calls with the modern fetch() based call, and also removed a few annoyances of old library. 

I've use trubolink before; and have always liked it. However, I've come to ask myself several times: Is Turbo drive still necessary? After all, it provides no functionality other than improvement of user's experience. How do we quantify the user experience improvement?

If we group the web page loadings into 2 classes: fast ones (both network and server response) and slow ones. For the fast webpages, with the latest Firefox and Chrome browsers, there is really no difference at all that I can feel. The transition of webpages, even without turbo drive, is smooth enough. With Safari or older browsers though, there is still a user perceivable difference: If you click a regular link, you will see a blank window for a brief moment. 

If you have a slower network link or the server is not resonding fast enough, there will be a significant difference, with any browser. Without turbo drive, the window will go blank until the webpage is loaded. With Turbo drive though, it will be different, depends on whether turbo drive has seen the URL before or not:

If turbo drive has seen the URL and has a version of the webpage in its cache, it will show the old version instantly, while loading the new one in the background. Once the new one loads, the browser window will switch. The catch here is while the user see something very fast, they cannot interact with the webpage right away, because it is the wrong one. Also it will create a "deja vu" effect if the old version and the new version look similiar but are different. 

If turbo drive has not seen the same URL before, it has no way to trick you with a cached web page. Instead, there will be a brief moment of lag: the page you are navigating away from is still shown after the click. Then turbo will paint a progress bar while keep showing the page you are navigating away, until the new page is loaded in the background and swtiched to.

Over the last two years, Firefox and Chrome has became faster, so that in the fast cases they render turbo drive largely obsolete. In the slow cases, the effect of turbo drive may appeal to some users but is also unexpected to the mojority of web users. Is my connection slow, or my browser acting wierd, or my mouse is broken? Without turbo drive, at least this part is clear.

Also I need to point out that turbo drive save neither network bandwidth, nor server loads. The same webpage is transferred, parsed, and laid out by the browser. If anything, it adds works to the browser because the possible double rendering.

## Turbo Frame ##

Turbo frame is brand new:

> Turbo Frames allow predefined parts of a page to be updated on request. Any links and forms inside a frame are captured, and the frame contents automatically updated after receiving a response. 

Normally, with a server-side render page resulting from you clicking a link or submitting a form, the server send you a full new page. Often times, the new page contains similiar content as the old page, with only a part of it updated. Yet still a full page need to be transferred, parsed, and laid out. This is obviously wasteful. So recently SPAs (single page application) use javascript to do a API call to grab data in JSON form, then use the returned JSON data to update the page locally. This is much more efficient, however the catch is now you have to have significant logic in the javascript, which add to the complexity.

Turbo frame formulates a protocol so the client can request part of a page based on user action and page markup, and the server will send just this part of the page, and the client will only update this part of the page. You reduced overhead, together with a simple and standard client side logic. 

However, I find the protocal still limiting. First of all, the part to be updated, the "turbo-frame", is pre-determined by the markup. So, you can only update one thing for one action, and it is not changable in run time. Secondly there is no formal way handle failed actions.

Inspired by turbo frame, I coded up something different. Again, some GET or POST routes in my server are actually partial page updates that return only snippet of HTML, not a full page. What is different is the target of the update is not hard coded: instead, the server side picks the target at run times, depending on situations. The target element id is sent through with a custom HTTP header, so the client side can follow orders. I've also abuse the HTTP response code a little to encode more meta data:

* 200 OK. Just replace the node cotent.
* 201 CREATED. The node content is cleared, and the HTTP reponse body contains a message to be displayed at the info flash.
* 202 ACCEPTED. The node content is not replaced, and the HTTP reponse body contains a message to be displayed at the info flash.
* 400 BAD REQUEST. The node content is not replaced, and the HTTP reponse body contains a message to be displayed at the error flash.

Custom header and custom HTTP response code can be done in any server-side framework; In [Phoenix](https://www.phoenixframework.org/) it is super easy:

``` elixir
    conn
    |> put_layout(false)
    |> put_resp_header("frame-target", target)
    |> put_status(:bad_request)
    |> text(msg)
```

On the client side it is also simple enough to implement:

``` javascript
function partial_request(request) {
    clear_info();
    clear_error();
    show_progress_bar();
    fetch (request)
	.then(response => {
	    var target = response.headers.get("frame-target");
	    var container = document.getElementById(target);
	    if (response.status == 200) {
		// ok. clear info and error, replace original html
		response.text().then(text => {
		    hide_progress_bar();
		    container.innerHTML = text;
		    window.document.dispatchEvent(new Event("DOMContentLoaded", {
			bubbles: true,
			cancelable: true
		    }));
		});
	    } else if(response.status == 201) {
		// created. set info message, clear original html
		response.text().then(text => {
		    hide_progress_bar();
		    flash_info(text);
		    container.innerHTML = "";
		    window.document.dispatchEvent(new Event("PartialRequestCancelled"));
		});
	    } else if(response.status == 202) {
		// accepted. set info message, keep original html
		response.text().then(text => {
		    hide_progress_bar();
		    flash_info(text);
		    window.document.dispatchEvent(new Event("PartialRequestCancelled"));
		});
	    } else {
		// bad request. set error message, keep original html
		response.text().then(text => {
		    hide_progress_bar();
		    flash_error(text);
		    window.document.dispatchEvent(new Event("PartialRequestCancelled"));
		});
	    }
	});
}
```

I did not include the full code but it is vastly smaller than Turbo. And it suits my needs. If you go over to [roastidio.us](https://roastidio.us) you can see plenty of this kind of partial loading in action. 

## Turbo Stream ##

Turbo Stream is also brand new:

> Turbo Streams deliver page changes as fragments of HTML wrapped in self-executing <turbo-stream> elements. Each stream element specifies an action together with a target ID to declare what should happen to the HTML inside it.

It is basically turbo frames with more elaborated action verbs, and are delieved via web socket. With a regular HTTP conversation, there can only be one response for each request, and you cannot have unsolicited responses. There are only so much you can do with Turbo frames. With web socket, you can do anything you want from the server side, send any number of async responses, so long as the client side can intepretate and act upon them.

However, with web socket, you have to have a persistent server side component, which is some cost to consider. And if I am willing to pay this cost, why don't I do one step further, and make the server-side stateful and taking full charge of the actions? This is percisely what [Phoenix Live view](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html) is.

## Summary ##

I end up not using any of the Turbo drive, Turbo frames, or Turbo stream because:

* Turbo drive is simple, however it is either unnecessary or confusing to the users. Also it saves nothing resource wise.
* Turbo frame is cool, however a competent programmer with full control of both the server side and the client side can achieve similiar results with far less code
* Turbo stream is the mose advanced and most powerful of the three, however we already have something even more advanced and powerful in Phoenix Live View.
