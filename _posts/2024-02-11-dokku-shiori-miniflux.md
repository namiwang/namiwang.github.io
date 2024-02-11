---
layout: single
title: 'Shiori and Miniflux on Dokku as Self-hosting RSS Reading Solution'
date: 2024-02-11 21:28:00 +0800'
classes: wide
---

My first New Year's cyber resolution for 2024 is to migrate more services and data to self-hosting solutions.

I already have a server with dokku for all my side projects, code-name hatchbox. I migrated projects from heroku to dokku years ago and I've been very happy with it.

[Shiori](https://github.com/go-shiori/shiori) is a simple bookmarks manager; [Miniflux](https://github.com/miniflux/v2) is a minimalist RSS reader. Both simple and well-maintained.

{% include figure.html
  src="2024-02-11-dokku-shiori-miniflux/shiori.png"
  alt="Sample shiori site"
  caption="Sample shiori site"
  maxwidth="480px"
%}

{% include figure.html
  src="2024-02-11-dokku-shiori-miniflux/miniflux.png"
  alt="Sample miniflux site"
  caption="Sample miniflux site"
  maxwidth="480px"
%}

After hosting both, I can configure miniflux calling shiori API to enabling marking of articles in one click.

{% include figure.html
  src="2024-02-11-dokku-shiori-miniflux/shiori-config-miniflux.png"
  alt="Miniflux with shiori integration"
  caption="Miniflux with shiori integration"
  maxwidth="320px"
%}

## translate docker-compose config to dokku commands

One thing I'm not entirely satisfied with is that dokku [still doesn't support docker-compose](https://github.com/dokku/dokku/issues/5102) natively.

Docker-compose offers a declarative approach. [Here](https://github.com/miniflux/v2/blob/main/contrib/docker-compose/basic.yml)'s a sample configuration for Miniflux.

I've annotated it with approximate equivalent dokku commands.

```yaml
services:
  miniflux:
    image: ${MINIFLUX_IMAGE:-miniflux/miniflux:latest} # dokku git:from-image
    container_name: miniflux # dokku app:create miniflux
    restart: always
    ports:
      - "80:8080" # dokku domains:add
    environment:
      - DATABASE_URL=postgres://miniflux:secret@db/miniflux?sslmode=disable # dokku postgres:link
      - ADMIN_USERNAME=admin # dokku config:set for general env vars
      - ...
  db:
    image: postgres:15 # dokku plugin:install postgres
    container_name: postgres # dokku postgres:create
    environment:
      ...
    volumes:
      - miniflux-db:/var/lib/postgresql/data
volumes:
  miniflux-db:
```

On the other hand, dokku relies on imperitive commands to deploy apps, the above docker-compose config can be roughly translated into commands:

```shell
# create and config app
dokku apps:create miniflux
dokku config:set --no-restart miniflux ADMIN_USERNAME="admin"
dokku git:from-image miniflux miniflux/miniflux:latest # `git:from-image` sets app source to docker image instead of git repo

# create and link db
dokku postgres:create miniflux
dokku postgres:link miniflux miniflux # this automatically sets DATABASE_URL env var

# port mapping
dokku ports:add miniflux http:80:8080

# (optional) domain and cert
dokku domains:add miniflux miniflux.some.domain
dokku letsencrypt:enable miniflux
```

As long as the services in your `docker-compose` are popular enough to be covered by [dokku plugins](https://dokku.com/docs/community/plugins/), you should be able to translate the configuration.

There's already a tool [dokkupose](https://github.com/stockmind/dokkupose/) to streamline this process. Although it has been unmaintained for 2 years - so [some dokku commands are already deprecated](https://github.com/stockmind/dokkupose/issues/2) - it still provides a very good starting point for you to tweak.

{% include figure.html
  src="2024-02-11-dokku-shiori-miniflux/dokkupose.png"
  alt="commands translated by dokkupose"
  caption="commands translated by dokkupose, almost ready to use."
%}

I feel like an LLM could do this translation better than me, so I'll end with a reminder:

- [ ] test docker-compose to dokku translation with LLM.
