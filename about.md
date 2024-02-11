---
layout: single
title: About & Projects
permalink: /about/
---

I write different kinds of code.

I participated in competitive programming as a teenager (NOI '08, '09 in China), and now I work as a full-stack developer for @microsoft.

I've been working on indie games, compilers, and regular full-stack developments.

I also translate tech materials and write [short stories](/tags/#story).

# side-projects

## fiber note

<details markdown=1>
<summary markdown="span">a bi-directional networked note-taking app</summary>

<a href="https://github.com/namiwang/fiber-note" target="_blank">
  <img src="/assets/images/fiber-note.gif" width="480" alt="fiber note screenshot">
</a>

[fiber-note](https://github.com/namiwang/fiber-note) is a bi-directional networked note-taking app which is open and self-hosting, inspired by `roam-research`, `obsidian` and others.

I'm working on a series of dev diaries around the building of fiber-note:

- [part #1](https://namiwang.github.io/2020/11/12/building-a-roam-like-networked-heavily-customized-realtime-editor-part-1.html)

</details>

## references.wiki

<details markdown=1>
<summary markdown="span">tracking pop culture references</summary>

<a href="https://references.wiki" target="_blank">
  <img src="/assets/images/ref-wiki.png" width=480 alt="references.wiki screenshot">
</a>

I watched/played so many tv shows/video games that I have to build [references.wiki](https://references.wiki/) to organize and visualize pop culture references between tv shows, video games, fictional characters, etc.

I composed scrappers to fetch and parse the occurrences of pop culture references in different wikia (fandom) sites. Currently I’ve parsed around 5000 references between 1000 works/celebrities.

I built my own knowledge base to cross-reference sources like wikidata, google kg, and wolfram alpha. I implemented a customized editor for user to create new references between works.

The next step will be UGC workflow (I’m considering a git-based one), editing history and data visualization.
</details>

## ruby on rust

<!-- https://github.com/gettalong/kramdown/issues/155 -->
<details markdown=1>
<summary markdown="span">An implementation of ruby in rust</summary>

[https://github.com/ruby-on-rust/ruby-on-rust](https://github.com/ruby-on-rust/ruby-on-rust)

An implementation of ruby in pure rust, consists of (barely working) lexer and parser, and a no-go interpreter.

I started this project as a way to learn rust. Eventually, I've learned more than that, including lexer & parser, `ragel`, and a lot of ruby's hidden magic.

Lots of the grammar rules and AST conventions are ripped from the `parser` gem. For lexer, I adapted some rules from `parser` and ported from ragel-6 to ragel-7. For parser, I use `syntax-cli` as the parser generator.

</details>

## project yoru

<details markdown=1>
  <summary markdown="span">visual novel generator</summary>

  <img src="/assets/images/project-yoru-1.png" width=480 alt="project yoru screenshot">

  <img src="/assets/images/project-yoru-2.png" width=480 alt="project yoru screenshot">

  [Project yoru](https://github.com/project-yoru) is a set of projects to build cross-platform visual novel games from only assets and scripts, without coding.

  To build this project, I was using `polymer`, lots of `gulp` scripts, `phonegap build`, which are all practically dead by now \:\|
</details>

<br>

# tiny projects

- [icons.land](https://icons.land): to search icons from different packs like font-awesome, ionicons, etc.
- [shiori raycast extension](https://www.raycast.com/namiwang/shiori): raycast extension for [shiori bookmark manager](https://github.com/go-shiori/shiori/).
- [wiki flutter](https://github.com/namiwang/wiki-flutter): a wikipedia client in flutter
- [1-toolbox.com](https://1-toolbox.com): a public toolbox wiki.
- [streaming guide](https://streaming-guide.github.io): help to decide which streaming service (nf, hulu, disney+) should you subscribe to

# packages/libraries

- <img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/ruby/ruby-original.svg" width="16" />[acts_as_reactable](https://github.com/public-reactions/acts_as_reactable) : emoji reactions support for rails
- <img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/dart/dart-original.svg" width="16" />[actioncable_dart](https://github.com/namiwang/actioncable_dart): action cable websocket protocol ported to dart
- <img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/dart/dart-original.svg" width="16" />[flutter_html_widget](https://github.com/namiwang/flutter_html_widget): parse HTML dom into flutter widgets

# publications

## as a translator

- I’m a co-translater of the simplified Chinese version of *MongoDB: The Definitive Guide* from *O'Reilly*

<!-- ## as a novelist

- I publish short stories at 圍爐（[weilu.community](https://weilu.community)） -->

<!-- # communities

- ingress enlighten lv.10
- help organized wikipedia offline events in shanghai -->
