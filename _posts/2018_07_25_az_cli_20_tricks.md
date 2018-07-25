---
ID: 472
post_title: >
  Some Az Cli 2.0 bulk commands
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2018/07/25/az-cli-20-tricks/
published: false
post_date: 2018-07-02 12:35:17
---
# Az CLI 2.0

## How to delete multiple managed disk at once ?
```bash
az disk list -g YourRG --query [].name --output tsv | xargs -n 1 az disk delete -g YourRG --yes -n
```
