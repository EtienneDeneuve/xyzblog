---
ID: 386
title: >
  Os Factory Azure et Aws en Open Source
  avec Microsoft et Société Générale !
author: etienne.deneuve
post_excerpt: ""
layout: layouts/post-sidebar.njk
mySlug: os-factory-azure-et-aws
permalink: "{{ page.date | date: '%Y/%m/%d' }}/{{ mySlug }}/index.html"
published: true
date: 2018-03-28 12:25:01
---
Nous venons de publier le deuxième provider (Azure ;)) dans notre OS Factory qui permets de générer des images pour AWS et Azure ainsi que de les partager entre deux souscriptions Azure. Il reste encore du travail, mais elle est fonctionnelle et cerise sur le gateau, c'est en Open Source!

Un grand merci à <span style="display: inline !important; float: none; background-color: transparent; color: #555555; cursor: text; font-family: 'Lato','Helvetica Neue',Arial,sans-serif; font-size: 18px; font-style: normal; font-variant: normal; font-weight: 300; letter-spacing: normal; line-height: 25.71px; orphans: 2; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;">la Société Générale (Yannick Neff), Microsoft France (Mandy Ayme),</span> <a href="https://www.wescale.fr/">WeScale</a> (Maxence Maireaux) et <a href="https://www.cellenza.com/fr/">Cellenza </a><span style="display: inline !important; float: none; background-color: transparent; color: #555555; cursor: text; font-family: 'Lato','Helvetica Neue',Arial,sans-serif; font-size: 18px; font-style: normal; font-variant: normal; font-weight: 300; letter-spacing: normal; line-height: 25.71px; orphans: 2; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;">(moi)</span><span style="display: inline !important; float: none; background-color: transparent; color: #555555; cursor: text; font-family: 'Lato','Helvetica Neue',Arial,sans-serif; font-size: 18px; font-style: normal; font-variant: normal; font-weight: 300; letter-spacing: normal; line-height: 25.71px; orphans: 2; text-align: left; text-decoration: none; text-indent: 0px; text-transform: none; -webkit-text-stroke-width: 0px; white-space: normal; word-spacing: 0px;">.</span>

Le code est par ici :

<a href="https://github.com/societe-generale/os-factory">https://github.com/societe-generale/os-factory</a>

Coté techno, nous utilisons Ansible et Packer. Il faut effectuer un petit setup coté Azure, il vous faudra une VM Ubuntu, avec le MSI de configurer et un SPN, tout est dans le <a href="https://github.com/societe-generale/os-factory#infrastructure-for-azure">readme </a>:)