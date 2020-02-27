---
ID: 120
title: 'Vorlon.js : Un petit billet pour un gentil copain dev !'
author: etienne.deneuve
post_excerpt: ""
layout: layouts/post-sidebar.njk
mySlug: vorlon-js-un-petit-billet-pour-un-gentil-copain-dev
permalink: "{{ page.date | date: '%Y/%m/%d' }}/{{ mySlug }}/index.html"

published: true
date: 2017-02-02 23:21:19
---
J'ai un copain dev' (pas qu'un) mais celui ma demandé il y a quelques jours:
<ul>
 	<li>Lui : "Tu sais, tu m'a parlé de Vorlon.js, c'est cool mais comment on change le binding, localhost m'en tape un peu"</li>
 	<li>Moi : "Bouge pas, je te donne ca"</li>
 	<li>Lui : "Y disent qu'il faut Nginx etc, (bref un gentil dev web en panique)"</li>
 	<li>Moi : "Mais non mais non"</li>
</ul>
Bon déjà Vorlon.js c'est quoi ? C'est un outil plutot orienté Dev, pas que web, fait par une bande de dev chez Microsoft, dont David Catuhe, Etienne Magraff, Julien Corioland, David Rousset et pleins d'autres que vous pouvez trouver (je vous mets pas tout ici, ca ferait trop de noms !) <a href="https://github.com/MicrosoftDX/Vorlonjs/graphs/contributors">ici</a> sur le github de Microsoft DX. Cet outil est une promesse :
<blockquote>A new, open source, extensible, platform-agnostic tool for remotely debugging and testing your JavaScript. Powered by node.js and socket.io.

Understand all about Vorlon.js in 20 minutes watching this video : <a href="https://channel9.msdn.com/Shows/codechat/046"><u><span style="color: #0066cc;">https://channel9.msdn.com/Shows/codechat/046</span></u></a></blockquote>
Deja, ca vient de Microsoft, moi j'aime bien (&lt;3 Linux aussi, pas de jaloux), le logo c'est un soucoupe mode retro gaming, ca peut que être fun, le site il est ici : http://vorlonjs.com/ . Bref, je l'installe sur mon mac pour dépanner le copain.

la doc me dit "nodejs", ok ben on commence par installer nodejs, depuis leur site : https://nodejs.org/en/download/

ca s'installe en mode IT Guys (le mauvais, pas le bon) "Click Next, Click Next, Click Finish"
<pre><span class="pl-s1">sudo npm i -g vorlon
sudo vorlon
<span class="pl-mo">With the server is running, open http://localhost:1337 in your browser to see the Vorlon.JS dashboard.
</span></span></pre>
Tiens comme la doc dis donc (et le copain dev du début)!
Je suis pas forcément un grand habitué de npm et de node en général mais je sais comment la commande find fonctionne donc je lance une petite recherche de vorlon qui se trouve dans /usr/local/lib/node_modules/vorlon et dedans je retrouve le dossier Server (que j'avais repéré dans le git avec un fichier de config en json qui s'appelle config.json. (limpide !)

Sachant qu'on cherche comment changer le binding de "localhost", je me dis "comment ont ils pu appeler le host", c'est host tout simplement, donc je change par mon fichier par  :
<pre>{
    "baseURL": "",
    "useSSLAzure": false,
    "useSSL": false,
    "SSLkey": "cert/server.key",
    "SSLcert": "cert/server.crt",
    "activateAuth": false,
    "username": "",
    "password": "",
    "host": "toto",
    "port": 1337,
    "enableWebproxy": true,
[...] }</pre>
je teste (oui c'est bien aussi les tests!)
<pre>sudo vorlon
With the server is running, open http://toto:1337 in your browser to see the Vorlon.JS dashboard.
</pre>
Bref, tout ça pour dire, franchement les gars, votre Vorlon.js c'est vachement cool !