---
layout: post
title: jekyll config
author: camh0we
excerpt: here's how this blog is configured
tags: jekyll github-actions deploy blog
---

# goal

simple jekyll site hosted via github pages

# config

- jekyll site running locally
- public github repository
- domain registered via cloudflare
- dns CNAME records pointing at repo's gh pages url
- CNAME file in root of repo
- gh action for jekyll deploy

```yaml
name: Build and deploy Jekyll site to GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  jekyll:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      # Use GitHub Actions' cache to shorten build times and decrease load on servers
      - uses: actions/cache@v2
        with:
          path: vendor/bundle
          key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile') }}
          restore-keys: |
            ${{ runner.os }}-gems-
      # Standard usage
      - uses: helaili/jekyll-action@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          target_branch: "gh-pages"
```

- jekyll `_config.yml`

```
repository: camh0we/h0weblog
baseurl: ""
url: https://camh0we.com
```
