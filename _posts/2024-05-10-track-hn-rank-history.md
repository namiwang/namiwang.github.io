---
layout: single
title: 'Track HN: Rank History and My Nocode Experiences'
date: '2024-05-10 19:00:00 +0800'
classes: wide
---

I've been working on the project [Track HN](https://www.track-hacker-news.com) for a while now. The HN dataset is perfect to play with. It's a vivid engaging forum about interesting topics from a community of hackers, with a ranking algorithm plus human interference. It's not so large as to require impossible resources for me to process. It just has so much potential.

I'm constantly expanding my dataset by tracking more data. Last year, I started to detect and track availability of links. Since April, I've begun to track score and rank history of all stories.

My next step is an API to serve this data and maybe an extension for the HN site.

I'm considering how to analyze the content - topics, semantic, sentiment, etc. - of stories and comments. I can't help but be intrigued about (anonymized for sure) user behaviors and profiles as well. I also want to explore other data analysis/visualization solutions.

If you have any interesting ideas, feel free to [let me know](mailto:me@nami.land).

## more data tracked: score and rank history

{% include figure.html
  src="2405-track-hn/story-history-chart-demo.png"
  alt="Story data with score and rank history"
  caption="Story data with score and rank history"
  maxwidth="480px"
  url="https://track-hacker-news.com"
%}

With the historical data on score and rank, I can analyze how these values have changed over time. This allows me to compare the performance of each story with others.

## dashboards

I've built dashboards with retool to help analyze the data. It started as a self serving tool, but I think it could be useful for others too so I made it public.

### [stories data overview](https://namiw.retool.com/p/track_hn-stories)

{% include figure.html
  src="2405-track-hn/track_hn_retool_stories_screenshot.opt.png"
  alt="Stories Data Overview"
  caption="Single Stories Data Overview"
  maxwidth="640px"
  url="https://namiw.retool.com/p/track_hn-stories"
%}

### [single story analysis](https://namiw.retool.com/p/track_hn-story)

{% include figure.html
  src="2405-track-hn/track_hn_retool_story_screenshot.opt.png"
  alt="Single Story Analysis"
  caption="Single Story Analysis"
  maxwidth="640px"
  url="https://namiw.retool.com/p/track_hn-story"
%}

## good experiences with retool

- The editor is good. It's a WYSIWYG editor that really just works <sup>TM</sup> most of the time, which is quite an impossible task. The overall building experience is smooth and intuitive. The built-in console helps.
- The components library covers most cases. For data-related components, it's easy to connect to different sources. The editor usually guesses the data source and type correctly.
- Querying data is a breeze. SQL interpolation with `{%raw%}{{ }}{%endraw%}` is quite useful (thanks for not inventing a new query language). Queries can be throttled/debounced. The results can be easily transformed and cached.
- It's battery-included. It handles common scenarios like url params and users authentication. It's possible to share the app and collab with other editors.
- The AI copilot in the SQL editor is helpful since it already has the context of the tables and schemas.
- The hosted Postgres is a nice additional feature. You can easily get a working connection string in an instant.

## not-so-good experiences with retool

### responsive is non-existent

Basically, in retool the canvas is not flow-based and mobile-first is not a thing.

Yes, there's one "mobile layout" option for application, but it's not what you'd imagined. It's blank by default and requires you to manually toggle the "show on mobile" option for every component. The components don't maintain the same order in mobile view either. It took days for me to realise that: every component simply have two separate positions configured: one in desktop view and one in mobile view. **That's not responsive at all, that's simply two views** you have to maintain.

Sometimes, I get the "component intersects with another component" error when adding component to mobile view, when they actually don't intersect with each other.

There [are](https://community.retool.com/t/mobile-responsive-design-in-retool/21791/) [many](https://community.retool.com/t/make-show-on-mobile-true-by-default/4733/) [complaints](https://community.retool.com/t/button-to-toggle-on-off-all-show-on-mobile-show-on-desktop/5112) about this problem. They've been teasing about a Retool Mobile solution, but it has been a long time and it couldn't fix current apps.

A [user even wrote a script](https://community.retool.com/t/enabling-mobile-for-whole-page/22804) to toggle the mobile option for all components (sadly it doesn't work anymore without hacking with DOM). When your user write code to fix your product, you know you have a problem.

Doesn't have mobile view is one thing, but cannot fallback to view the desktop version on mobile is worse. It's impossible to pinch/zoom on mobile even after "request desktop site" on mobile browsers. I guess it's about viewport configuration but haven't dig into it.

### custom components are not working

Custom components looks much better than modules. However, in my case it doesn't render properly. There's already [complaints](https://community.retool.com/t/custom-components-dont-render-when-viewing-via-public-link/39593) but with no update.

## features I wish retool (and other platforms) had

### building

- Please provide a way to reveal the code. Imagine a code/GUI mode toggle for every component and even on the app level. There's already import/export feature with seraialized data for the whole app. So why not just let me edit directly instead of export, batch edit, and import back?
- Auto-generated admin dashboard from database. It makes sense since you already have the schema. What's better than nocode? No action required at all.
- An LLM-Copilot is actually not a bad idea for nocode app generation and modification (I'll bet there's already a prototype).

### querying

- It'd be nice to pass an object (say an input component) instead of only string values into a shared query.

### components

- I hope there's a library agnostic chart component. In many cases, before using a nocode platform, the user might already had a preferred chart lib and tons of working data visualization configurations.

### database

- The hosted Postgres is nice but it lacks administration features. I can live with no performance dashboard since I can query the data on my own with other tools. But features like upgrading, backup/restore and replication configuration are must-have for serious use.

## when not to nocode

Basically, you don't go no-code when you actually need the advantages offered by the code.

Codes are easier to edit, share, reuse, track, test and analyze, at least for some users. Good codes can add unambiguous complex logic. At this point, I feel nocode is still only good for prototyping simple or internal apps.
