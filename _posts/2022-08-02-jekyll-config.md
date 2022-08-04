---
layout: post
title: jekyll config
author: camh0we
excerpt: here's how this blog is configured
tags: jekyll github-actions deploy blog
---

## goal

simple jekyll site hosted via github pages

## config

- jekyll site running locally
- public github repository
- domain registered via cloudflare
- dns CNAME records pointing at repo's gh pages url
- CNAME file in root of repo
- gh action for jekyll deploy

jekyll `_config.yml`

```
repository: camh0we/h0weblog
baseurl: ""
url: https://camh0we.com
include: [CNAME]
```
