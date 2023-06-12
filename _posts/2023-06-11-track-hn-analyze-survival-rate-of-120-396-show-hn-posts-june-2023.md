---
title: 'Track HN: analyze the survival rate of 120,396 Show HN posts (June 2023)'
excerpt: 'Spoiler alert: many of the good projects didn''t survive'
date: 2023-06-11 21:00:00 +0800'
---

## The HN database

I maintain an up to date database of all public HN items (links, comments, users, etc.).

### Data source

- Public dataset. The Google Cloud BigQuery public datasets include [one for Hacker News data](https://console.cloud.google.com/marketplace/details/y-combinator/hacker-news). However, this dataset has not been updated since November 2020.
- [Official api](https://github.com/HackerNews/API). I use this to update my database constantly.

### Numbers

For this analyze, I considered submissions made before May 31, 2023, 23:59 UTC. The dataset consists of 4,714,023 stories and 30,363,533 comments from 867,097 users.

## Show HN stories

### Overview

There are 120,396 valid Show HN stories. The [first one](https://news.ycombinator.com/item?id=510264) was submitted on March 10, 2009, and [the last one](https://news.ycombinator.com/item?id=36145942) was submitted on May 31, 2023, at 23:38 UTC.

### Submissions yearly trend

<a href="https://graphy.app/view/1079280c-cc72-4c04-aaa3-7a87825b8d83" target="_blank">
  <img src="/assets/thn/2306/Show HN submitted by year.png" alt="Show HN submitted by year">
</a>

### Submissions monthly trend

<a href="https://graphy.app/view/74f719aa-8b65-4948-8a16-5756561c61a6" target="_blank">
  <img src="/assets/thn/2306/Show HN submitted by month.png" alt="Show HN submitted by month">
</a>

### Heatmap (week x hour) TBD

<img src="/assets/thn/2306/show-stories-heatmap.svg" alt="Show HN submissions heatmap">

### Same heatmap but for top 1% stories

<img src="/assets/thn/2306/show-stories-heatmap-for-top-1-percent-items.svg" alt="Show HN submissions heatmap top 1%">

### Top domains used

{% assign data = site.data.thn.2306.show-hn-stories-group-by-domain %}
<table>
  {% for row in data %}
    {% if forloop.first %}
    <tr>
      <th></th>
      {% for headers in row %}
        <th>{{ headers[0] }}</th>
      {% endfor %}
    </tr>
    {% elsif forloop.index0 <= 20 %}
    <tr>
      <td>{{ forloop.index0 }}</td>
      <td>{{ row.domain }}</td>
      <td>{{ row.count }}</td>
    </tr>
    {% endif %}
  {% endfor %}
</table>

<details>
  <summary>More data</summary>
  <table>
    {% for row in data %}
      {% if forloop.first %}
      <tr>
        <th></th>
        {% for headers in row %}
          <th>{{ headers[0] }}</th>
        {% endfor %}
      </tr>
      {% elsif forloop.index0 > 20 %}
      <tr>
        <td>{{ forloop.index0 }}</td>
        <td>{{ row.domain }}</td>
        <td>{{ row.count }}</td>
      </tr>
      {% endif %}
    {% endfor %}
  </table>
</details>

#### Extra: ChatGPT gave a wrong regex

I consulted ChatGPT for a regex to extract domains from urls, and it gave a flawed one:

> `^(?:https?:\/\/)?(?:[^@\n]+@)?(?:www\.)?([^:\/\n?]+)`.

It even gave reasonable detailed explanations which convinced me. Later tests revealed that this regex doesn’t work for url with `@` in path, such as `https://foo.com/@./bar`. The correct one should be

> `^(?:https?:\/\/)?(?:[^@\/\n]+@)?(?:www\.)?([^:\/?\n]+)`.

### Show HN stories survival rate

In total: out of all 120396 valid Show HN stories, 79,999 (66.45%) survived.

<a href="https://graphy.app/view/24349bfe-936f-4078-ac4f-94aee89c13ab" target="_blank">
  <img src="/assets/thn/2306/Show HN stories survival rate.png" alt="Show HN stories survival rate">
</a>

#### Notes about survival rate calculation

I calculate the survival state by checking the status of the submitted url. The calculation might not be accurate due to many reasons:

- False positive cases (dead projects counted as live):
  - The project has retired and now its domain is being used for other purposes. (spam site, redirecting to domain register, a whole new project, etc.)
  - The project is practically dead but still accessible (e.g. an archived code repository), and it’s debatable whether to count these as dead.
- False negative cases (live projects counted as dead)
  - The site might only be temporarily inaccessible.
  - The site might refuse to serve my detector:
    - It might determine my request as from a malicious crawler. I use a Cloudflare worker which doesn’t behave like a real user.
    - It might only serve users from specific regions, thus showing an error for me.
  - The site might return HTTP 418 as a joke (e.g. [https://coneapp.io](https://coneapp.io/))
    - (I’ve corrected this one and now count 418 as a live response, only want to mention it because of its humorous and surprising nature.)

### Individual project survival rate (excluding articles and code repositories)

To prioritize individual projects over news, articles, and code repositories, I created a new query by filtering out domains that appeared more than three times in all Show HN stories.

With this constraint applied, out of 64407 Show HN projects, 34585 (53.70%) survived.

<a href="https://graphy.app/view/96d508a2-2f06-4566-8c6c-85ca6663a789" target="_blank">
  <img src="/assets/thn/2306/Show HN stories survival rate (individual domains).png" alt="Show HN stories survival rate (individual domains)">
</a>

### Top-scoring Show HN stories that didn’t survive

{% assign data = site.data.thn.2306.show-hn-stories-top-score-didnt-survive %}
<table>
  {% for row in data %}
    {% if forloop.first %}
    <tr>
      <th></th>
      <th>title</th>
      <th>score</th>
      <th>submitted at</th>
    </tr>
    {% elsif forloop.index0 <= 20 %}
    <tr>
      <td>{{ forloop.index0 }}</td>
      <td>
        <a target="_blank" href="https://news.ycombinator.com/item?id={{row.id}}">{{ row.title }}</a>
      </td>
      <td>{{ row.score }}</td>
      <td>{{ row.time | date: "%b %Y" }}</td>
    </tr>
    {% endif %}
  {% endfor %}
</table>

<details>
  <summary>More data</summary>
  <table>
    {% for row in data %}
      {% if forloop.first %}
      <tr>
        <th></th>
        <th>title</th>
        <th>score</th>
        <th>submitted at</th>
      </tr>
      {% elsif forloop.index0 > 20 %}
      <tr>
        <td>{{ forloop.index0 }}</td>
        <td>
          <a target="_blank" href="https://news.ycombinator.com/item?id={{row.id}}">{{ row.title }}</a>
        </td>
        <td>{{ row.score }}</td>
        <td>{{ row.time | date: "%b %Y" }}</td>
      </tr>
      {% endif %}
    {% endfor %}
  </table>
</details>

## Next steps

### Send me your interesting queries

If you have some interesting queries in mind, feel free to ping me (TBD)!. Maybe you’re preparing to launch your own product, or conducting research, or preparing data for your own ML model, or just messing around, I’d like to hear your ideas! I’ll publish reports/queries with proper credits.

### Looking for a sponsor to host the database publicly

In the meantime, it’d be great if anyone can query the database. I tried to host a public database and real-time query interface online, but couldn’t afford the bill for a smooth Postgres instance to hold around 20G (40M rows plus indices) data. Yes, a $20 instance would do the job but it’s pretty slow from usable, comparing to the local one on my M2 MacBook Air.

<img src="/assets/thn/2306/dashboard_demo.png" alt="demo data dashboard">

So I’m looking for a sponsor to host the database publicly. I need one mediocre VM for a Rails stack app and a semi-powerful hosted Postgres instance. Contact me at TBD if you’re interested.
