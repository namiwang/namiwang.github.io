---
title: I published my toolbox as one public database
layout: single
excerpt: Now I use 1-toolbox.com
date: 2022-03-22 18:18:18 +0800
---

<a href="https://1-toolbox.com" target="_blank">
  <img src="/assets/images/1-toolbox.png" width="640" alt="1-toolbox.com">
</a>

## it's hard to remember these names

For years when I run into useful tools or services, I save them for later. There're several issues with this approach:

- They're scattered in browser bookmarks, github stars, and random tweets.
- It's hard to find the right one when I need something I vagualy remember marked. I often forget the name of the product. So I need to query by industry and form.
- Some of them are not available anymore.

I want to have a small system to give me answers to questions like:

- Ah what's that self-hostable service to deal with webhooks with official ruby sdk that someone mentioned weeks ago?
- Ouch this public api is not working anymore I think there's another similar one?
- Hey what did we use to draw diagrams for our last year's keynote? You know the one with lines, arrows, uh, lines? No?

## data to query

I defined several dimensions used to find the right tools:

- form (utility, service, api, etc.)
- tags (suit for everything like industry, features)
- self-hostable
- audience (developer, designer, scholar, etc.)
- officially supported languages

## I don't want to use notion, but it works

So I moved my bookmarks into a database on notion and shared it to the web. I'm stilling working on enriching the data and adding my marked/stared projects on other platforms.

I was reluctant to use notion primarily because:

1. It's proprietary and requires an account to edit.
2. The notion's database is just a wrapper around its block system, and a record is just a page block with several more properties. There's no real cell and you can't comment on or track a specific cell.
3. The history/permission systems are basic level. There'd be no way to put out an <a href="https://en.wikipedia.org/wiki/Wikipedia:Edit_warring" target="_blank">edit war</a>.

But to be honest, notion is still the best way I've found to share a bunch of structured data while enabling others to edit and query.

It cost me less than 30 minuts to share the database, config one custom domain, and setup a real domain (🧰 <a href="https://1-toolbox.com" target="_blank">1-toolbox.com</a>) forwarding to the page.

My other choices are:

### wikimedia

It'd be a real wiki, open and anonymously editable. But wikimedia's system is built around articles, not structured records.

Of course there's wikidata. I'm not an expert but I think wikidata also use wikimedia, and the data are just wrapped articles. I don't know php and don't want to spend time on hacking it. However, I do think it's an promising system and would spend some time to experiment with it.

### google sheet

A public shared google sheet could also solve this. I tested it, and it's bloated and slow. Plus, a google account is necessary, so there's. The comment system is better, but I don't think it's possible to create different views (set of filters) without heavy hacking.

## the rules are simple but hard

The questions about abusing/censoring were exploding in my mind. So I decided to forget all of them and let it run wild for a while.

I set the edit permission to everybody, added some simple rules and am hoping for the best.

- don’t destruct
  - don’t mass delete or insert inappropriate content
- don’t poison
  - don’t insert malicious links or links with reference code/tracker
- don’t reap
  - don’t edit/comment in return for money or similar inducements, or delete your competitor
- don't brawl
  - don’t start, join, or escalate an edit war
- don't advertise
  - do give meaningful contribution before self-promote

## feedback welcome

Thanks for reading. If you find 
🧰 <a href="https://1-toolbox.com" target="_blank">1-toolbox.com</a> 
useful. Please help share it and maybe update the data as you see fit, eventually all of us can have a better toolbox.
