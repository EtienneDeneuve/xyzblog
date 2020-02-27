---
ID: 472
title: >
  Some Az Cli 2.0 bulk commands
author: etienne.deneuve
post_excerpt: ""
layout: layouts/post-sidebar.njk

mySlug: az-cli-20-tricks
permalink: "{{ page.date | date: '%Y/%m/%d' }}/{{ mySlug }}/index.html"
published: true
date: 2018-07-02 12:35:17
---
# Az CLI 2.0

## How to delete multiple managed disk at once ?
```
az disk list -g YourRG --query [].name --output tsv | xargs -n 1 az disk delete -g YourRG --yes -n
```
