---
layout: single
title: building a roam-like, networked, heavily-customized realtime editor, part 1
excerpt: I can build this. &mdash; every developer at least once
date: 2020-11-12 20:21:42 +0800
---

# about this project

> I can build this.
>
> &mdash; <cite>every developer at least once</cite>

Knowledge is hard to manage, as mind is hard to materialize and visualize.

Bi-directional networked tools like `roam-research` and `obsidian` are on the trend for a while now. [The idea behind them](https://en.wikipedia.org/wiki/Knowledge_graph) is not brand new, yet the much evolved web-based tech makes them possible.

## what I want to build

I record my building of `fiber-note` in this series of dev posts, what I want to build is:

* tag-based bi-directional networked note-taking
* web-based real-time experience
    * constantly auto-saving
    * real-time reactive interface
        * updating components other than the editor (related notes, navigation, tags network, calendar, etc.)
* a highly-customized editor
    * enforce a custom schema to control the layout of the documents
        * say everything is a list item, there may be only text and tags in a paragraph, etc.
    * handle meta-data like tags, assigning unique ids for the block
    * complex UI like inline drop-down menu
    * …
* visualize data as a network and a calendar
* open-sourced and self-hosted

## code & demo

<a href="https://github.com/namiwang/fiber-note" target="_blank">
  <img src="/assets/images/fiber-note.gif" width="480" alt="fiber note screenshot">
</a>

I placed the source [here](https://github.com/namiwang/fiber-note), which could be hosted by yourself. There’s even a configured deploy-to-heroku button for one-click hosting.

I also put up a public demo running at [fiber-note-demo.herokuapp.com](https://fiber-note-demo.herokuapp.com/session/new) (password is `password`).

## naming the project

I always use rails as my first choice for web projects, and the first code name for this project is `roam-on-rails`, which is a bad joke since I’ve already got a project called [ruby on rust](https://github.com/ruby-on-rust/ruby-on-rust).

I’m bad at this, so I just picked the word `fiber` as a synonym for `network` from the thesaurus.

# designing the data structure

## a prototype on paper

<a href="/assets/images/fiber-note-series/fiber-note-diagram.png" target="_blank">
  <img src="/assets/images/fiber-note-series/fiber-note-diagram.png" alt="fiber note data structure">
</a>

## a data structure in mind

The whole database is structured as a directed graph. The basic unit is a `block`, a node in the graph, representing a paragraph, bearing data like its content and optional tags.

Every block-node may have one parent. Thus a note, spawning from the root (a parent-free block-node), forms a tree and the whole database forms a forest.

There’re some edge cases to consider

* have to avoid cycles in the graph
* a tag may points to the same note

## a data structure on disk

Apparently we need to maintain a graph, yet I didn’t choose a graph-oriented database like `neo4j`. Using good old SQL to simulate one is good enough for now.

There’re both plugins to do graph on database-level ([AGE](https://www.postgresql.org/about/news/announcing-age-a-multi-model-graph-database-extension-for-postgresql-2050/) for postgresql), and app-level (e.g. [ancestry](https://github.com/stefankroes/ancestry)). For the initial implementation, I chose to hand-written everything from the grounded up for faster iteration because I was constantly changing things.

# choosing an editor

This is gonna be a front-end-heavy project, I have to choose an editor as one of the first steps.

## requirements

* restrict the doc to a special set of content types
    * e.g. allow list, list item, and inline tags; disallow individual paragraphs or images
* render existing data into the editor
* inspect and manipulate input from the user
    * enforce input rules like properly wrapping/indenting list items
    * assign unique ids to created blocks
* implement drop-down menu to auto-complete tags
* send updated content to the server
* features we may need in the future
    * copy-and-paste, drag-and-drop, image, etc.

## comparision

|	|trix	|quill	|prose-mirror	|
|---	|---	|---	|---	|
|integrating	|easy	|moderate	|moderate-to-high	|
|customization	|minimal	|moderate	|high	|
|custom schema	|no	|no	|yes	|
|plugins	|no	|yes	|yes	|
|themes	|no	|yes	|no	|
|docs	|minimal	|detailed	|detailed	|
|typescript	|no	|yes	|yes	|
|scenario	|adding rich-text editing to your rails app in 10 minutes	|building editor email client with some cool features like markdown syntax	|building editor for a collaborative encyclopaedia with custom schema.	|

I tried a few options. At the end I settled with [prose-mirror](https://prosemirror.net/) to build fiber-note due to thorough guides and references, up-to-date maintaining, and [a friendly forum](https://discuss.prosemirror.net/).

## trix

As a rails user, my first thought are `actiontext` and [trix](https://trix-editor.org/). Trix is perfect for adding out-of-box rich-text editing to a normal rails app, like rails, it just works.

It’s just hard to tweak for more custom features.

* it completely intertwined with rails’ components like `actionview` (rendering) and `activestorage` (image uploading), it’s hard to mutate the saved content without hacking into hidden methods.
* it saves content as raw HTML fragment, which is bad because
    * unique content have multiple legit representations
    * it’s slow to parse and manipulate the content (say tags detection, image processing, table mutation, etc.)
* it doesn’t come with detailed docs/specs about the format of generated HTML docs, which is not reliable when you system relies on processing the content on-the-fly.
* minimal events not enough to compose complex logic around the user's operation

## quill

[quill](https://quilljs.com/) is another competitive  candidate, regarding elaborated docs, data format specs, themes, and typescript support.

It’s easy to integrate the library, tweak some configurations, and apply different themes. You can add markdown syntax support in like 10 minutes.

It’s not easy to limit what kind of content is allowed in the document, or what would happen if a special formatted text is pasted.

It’s hard to pragmatically control the mutation of the data to manually implement functions like “create another list item with the same indent when *return* is pressed and current cursor on the end of a list item, including the end of an inline span of a list item”.

It’s not impossible, yet it will get over-complicated if you need many mutations like this.

## prose-mirror

`prose-mirror` has the most complicated structure.  You have to import **at least ten packages** to build a simple demo, each managing a single aspect of the editor (model, view, schema definitions, keymaps, etc.).

There’s a starter kit kind of package (`prosemirror-example-setup`) which makes the journey a little bit easier. I’d recommend that kit to users who demand only basic features, while for complex functions you’ll have to compose each part and piece them together for better control. It’s like how advanced users almost never want the pre-set default preferences or `rails scaffold`.

Verbosity and redundancy means total control and vice versa, it just have to be like this. You posses the ability to custom the schema, listen to each key press, oversee every transaction of the model, and to arrange how to create (also parse, and wrap, and truncate during drag-n-drop, etc.) different kind of element.

You’ll spend a lot of time jumping between the [guides](https://prosemirror.net/docs/guide/), the [references for individual packages](https://prosemirror.net/docs/ref/), and even the source code. I can promise you that the guide will be a great read about the designing of a complicated modular system.

# next chapter

Thanks for reading. In the next post, I'll discuss how I built and tweaked the editor.
