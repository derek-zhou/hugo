---
title: Search in Roastidio.us
slug: search_roastidious
date: 2021-08-30
tags:
- roastidious
description: Roastidio.us recently gained full text search capabilities! We can't challenge Google (yet), but now you can find interesting contents curated by you and your fellow roasters, from within Roastidio.us. 
---

[Roastidio.us](https://roastidio.us) lets you leave comments on pretty much anything: blog posts, news articles, podcasts, as long as they are publicly accessible. The search function is designed to solve the following 2 practical problems:

 * How do I find the interesting article that I stumble upon last month?
 * How do I find interesting contents that _other_ people is reading/roasting? 

## What do we index?

Unlike other search engines, Roastidio.us does not crawl the web at all. Every indexed document comes from member's inputs:

* We index every topic submitted by our members.
* We index everything contained in the RSS (or Atom/JSON) feeds loaded through us via the [AirSS](https://airss.roastidio.us) integration by our members.

We do not keep track of who was loading or reading what, so rest asured that your privacy is honored. Right now, there are only a few thousands documents indexed; and we do not expect the database to contain more than a few millions documents any time soon. So the amount of info indexed by roastitio.us is but a tiny fraction of the cyber space. However, we believe our search results can be much more relevant than what you can find on Google or elsewhere, because our index is curated by real persons, including you, so the signal to noise ratio should be way higher.

## How to use

Please click on the `FIND TOPICS` tab in your [portal page](https://roastidio.us/me) to find the search box. The query syntax is very simple: A query like `Joe Biden` means any document that contains `Joe` _or_ `Biden`. To seach for `Joe` _and_ `Biden`, you have to use `+Joe +Biden`. Or you can search for `"Joe Biden"`, which means any documents with `Joe` and `Biden` in consequtive positions. Any search term can also be limited to a particular field, eg: `title:Term` or `site:Term`. We distinguish 3 fields: title, body and site.

The results you see before you type anything in the search box are randomly selected. There is no secret ranking, paid inclusion or user profiling; we don't even know how. Please also excuse us and ignore the results should they offend you.

## Limitations

We do not offer search service to non-members. It would be of limited value to the non-members anyway, because they cannot influence the index and are better off using a generic search engine. 

Right now, we only index the topic (external web pages) but not your roasts. At roastitio.us, the visibility to individual roasts are different from member to member, and we do not want to violate that coveneant.

Lastly, there is no search hint when you type, and we do not suggest spelling correction. Roastidio.us search is very old fashioned; it will just faithfully search the index with the query you typed.

## Closing remarks

The search functionality is brand new, and may contain bugs. We encourage every member to use it and report issues. We also would appreciate every one to find interesting contents on the web and roast them in roastidio.us, so we can build up our index to be more useful to everyone. Another way to contribute is to use [AirSS](https://airss.roastidio.us) together with roastidio.us by loading feeds through roastidio.us. We can only be as useful as our members make us to be.

Roastidio.us search is built on the wonderful [Tantivy](https://github.com/tantivy-search/tantivy) full text search engine library.
