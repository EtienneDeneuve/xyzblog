---
ID: 55
title: >
  Get ink level from HP printers in
  Powershell
author: etienne.deneuve
post_excerpt: ""
layout: layouts/post-sidebar.njk
mySlug: get-ink-level-from-hp-printers-in-powershell
permalink: "{{ page.date | date: '%Y/%m/%d' }}/{{ mySlug }}/index.html"

published: true
date: 2016-01-15 09:58:06
---
I've find a way to get details about cardrige on low end hp printer !

Get it on my Git ! <a href="https://github.com/EtienneDeneuve/Powershell/blob/master/HpPrinter/Snippet" target="_blank" rel="noopener">HP Snippet </a>
<pre>$Web = New-object System.Net.WebClient
[xml]$stringprinter = $Web.DownloadString("http://printerhp/DevMgmt/ConsumableConfigDyn.xml")
$stringprinter.ConsumableConfigDyn.ConsumableInfo | Select ConsumableLabelCode,ConsumablePercentageLevelRemaining 
</pre>