---
title: How to write Javascript like Elixir
slug: how_to_write_javascript_like_elixir
date: 2021-03-22
tags:
- web
description: Javascript has a concurrent programming model that centers around promises, async functions and the await primitive. However, I want to use the conceptually simpler and more robust actor model that is widely used in the Elixir/Erlang world. Can I do it? Let's find out. 
---

I have a confession to make: I love the Elixir/Erlang concurrency model so I am biased. Recently I need to write a Javascript SPA (simgle page application) so I have to deal with the promise based APIs like [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API), [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) and 3rd party Web APIs. They feel awkward and racy to me; I cannot wrap my head arond all the `.then()...` calls and I was afaid that I would be making more bugs than I could fix. I want to use the more familiar and robust [Actor model](https://en.wikipedia.org/wiki/Actor_model), but could I?

## The Architecture ##

I break down my Javascript SPA into 3 layers: a frontend layer that handle the presentation layer logic, a backend layer that handles the business logic and a thin middle layer that bridge and decouple the frontend and the backend. The frontend is written in [Svelte](https://svelte.dev/), which I am not going to talk about in this article. The backend is written in vanilla Javascript without using any framework. From the outside, the backend is just a bunch of exported functions so the frontend client can query and manipulate application states. Besides function calls, it will also send notifications to the frontend. The notification part is easy: In elixir I would use [Phoenix.PubSub](https://hexdocs.pm/phoenix_pubsub/Phoenix.PubSub.html) and in Javascript I can just send a [custom event](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent/CustomEvent) to the global `Window` object. 

The backend layer screams [GenServer](https://hexdocs.pm/elixir/GenServer.html#content) to me.

## The GenServer model ##

GenServer is a design pattern widely used in the Elixir and Erlang world. It is a module of functions and a server process at the same time, and it has 2 sides:

* the client side, which runs in the client processes
* the callback side, which runs in a single server process

The clients will call the client side functions, and they in turn will send messages to the well known server process. The server process runs an event loop that dispatch all the messages in the received order to the callback side handlers. The messages could be one-way ( `casts` in Erlang speaks) if the client side do not expect a return value, or two-way (`calls`), if the client side is waiting for a returned value. For details, you can refer to the [official Erlang documentation on gen_server](https://erlang.org/doc/design_principles/gen_server_concepts.html)

The duty of a GenServer is to look after arbitrary state. Because the call back side runs in a single process, there is no race condition so the state are always consistent. Also all messages are handled in order to ensure causality. Deterministic results, consistent state, and observed causality make concurrent programming so much easier in the Elixir/Erlang world. Can I replicate the same pattern with Javascript?

## What do we have in Javascript ##

On the surface, Javascript concurrent programming cannot be more different from Erlang:

| Language | Elixir/Erlang | Javascript |
|----------|---------------|------------|
| threading |  multi-thread | single thread |
| scheduling | preemptive scheduling | cooperative yielding |
| communication method | message passing | function calling |
| Backlogs | message queue | promise chain |
| Data structure | immutable values | mutable variables |

What we have in Javascript are [async functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function). An async function looks like a regular function but is more like a [coroutine](https://en.wikipedia.org/wiki/Coroutine) that can yield and resume execution.  When called, it will yield at each `await` point, and return a promise. The promise is a handle to the in-flight coroutine, and the promises can be `await` -ed in turn. A promise will resolve when the async function run though its course. The re-scheduling of the outstanding coroutines are handled by the runtime automatically; so if one promise deep down is resolved the coroutines `await` -ing on it will become runnable and once resolved can trigger more outter coroutines. Eventually, everything will be resolved and we are back to the boring non-concurrent execution model of regular functions.

The problem here is the scheduling of runnable coroutines is uncontrollable and may even be non-deterministic, unless I have a strictly linear promise chain. 

## The Javascript GenServer ##

In my backend code, I have modules that behave almost like GenServers. Like a GenServer, the module has 2 sides: the client side and the callback side.

### The client side ###

The client side has exactly one shared variable: `state`, which is the top most promise. It must not be exported or accessed from the callback side. All the client side functions just wrap layers upon layers of promise on it:

``` javascript
let state = init();

// this is a one way cast
function doSomething(arg) {
	state = cb_doSomething(state, arg);
}

function doSomethingComplex(arg) {
	state = cb_doSomething(state, arg);
	state = cb_doSomethingElse(state, arg);
}

// this is a two way call
function getSomething(arg) {
	state = cb_getSomething(state, arg);
	return state;
}

function getSomethingExt(arg) {
    let ret = state = cb_getSomething(state, arg);
	state = cb_doSomethingElse(state, arg);
	return ret;
}

```

It is very important that the client side function shall not be `async` and shall not `await` on anything. All they do is to wrap promises, and optionally return a promise that may or may not be the top most one.

### The callback side ###

Each of the `cb_*` functions are the callback entry points, and they are all async functions that return promises. The promise could eventually resolve to a `null` as in the case of `doSomething`, or a value as in the case of `getSomething`. If the client side call `doSomething` then `getSomething`, they are guarantied to execute in that order, because inside the `cb_*` functions they `await` at the very beginning:

``` javascript
async function cb_doSomething(prev, arg) {
    await prev;
	...
}

async function cb_getSomething(prev, arg) {
	await prev;
	...
	return XXX;
}
```

The callback side can also have any number of module local variables as the state. They shall not be exported and has to be accessed by accessors that pass through a client side wrapper and a callback entry barrier. Each callback async function does not care what the previous promise is, only to `await` on it until it is resolved before doing anything so the whole callback side is not reentrant. 

To use an analogy, the promise chain is like [Russian dolls](https://en.wikipedia.org/wiki/Matryoshka_doll): The client side keep wrapping from the outside, and the call back side will resolve the promises from the inside out. This way, I achieved deterministic execution and infinite message queuing.

The client side can return one promise to the calling client to be `await` -ed on, or nothing at all. This way, I achieved the two-way calls and the one-way casts.

## Code Organization ##

As decreed by Saša in [To spawn, or not to spawn?](https://www.theerlangelist.com/article/spawn_or_not), processes, such as GenServers, shall only be used to seperate runtime concerns. In the beginning, I only have one GenServer, which serializes accesses to all the states. Later on, I splited some parts out to seperate GenServers, not because of business logic concerns, but because certain operations are slow and could benefit from running in a seperate GenServer. As of now I only have 2 additional GenServers:

* One to fetch network resurces that I need
* Another one to call a 3rd party web API (Airtable) that I use

The main GenServer still handles everything else, including the access of all the in-memory data structures and the access to the persistent backing store (IndexedDB). If I ever need to fetch network resources in parallel, or access another 3rd party web API, I would spin up more GenServers. The Javascript SPA is single threaded anyway, so the purpose of the concurency is only to hide latency.

## Summary ##

Through clever usage of the async functions and the await primitive, I achieved nearly the same design pattern of the Elixir GenServer. Please keep in mind that there is no process seperation and no supervisor, so if one part crashes, the whole thing crashes. Nevertheless, I believe this is something that can be useful to a broader audience. 

By the way, the project that I was working on is [AirSS](https://github.com/derek-zhou/airss), a client side only RSS aggregator. It is deployed [here](https://airss.roastidio.us); please give it a try. Feedbacks and pull requests are welcome.









