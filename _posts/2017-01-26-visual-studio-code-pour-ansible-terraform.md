---
ID: 97
post_title: '[Updated 20/03/2017] Visual Studio Code pour Ansible, Terraform'
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2017/01/26/visual-studio-code-pour-ansible-terraform/
published: true
post_date: 2017-01-26 22:42:55
---
L'Infrastructure as Code, c'est des méthodes et des outils pour faire des infrastructures reproductibles à l'infini.

J'utilise quelques outils sur mes deux postes de travail principaux : un pc sous Windows 10 et un mac sous macOs Sierra. J'ai donc besoin d'outils qui peuvent fonctionner sur les deux plateformes sans "galère" et efficace sur les deux plateformes.

J'utilise <a href="https://code.visualstudio.com/">Visual Studio Code</a> qui est gratuit, dispo sur Windows, Linux et macOs.

Voici quelques plugins que j'ai installé :
<ol>
 	<li>Ansible:
<ol>
 	<li><a href="https://marketplace.visualstudio.com/items?itemName=timonwong.ansible-autocomplete">Ansible-Autocomplete</a></li>
 	<li><a href="https://marketplace.visualstudio.com/items?itemName=haaaad.ansible"><span class="ux-item-name" data-bind="text: itemName">Language-Ansible</span></a></li>
</ol>
</li>
 	<li>Terraform:
<ol>
 	<li><a href="https://marketplace.visualstudio.com/items?itemName=mindginative.terraform-snippets"><span class="ux-item-name" data-bind="text: itemName">Advanced Terraform Snippets Generator</span> </a></li>
 	<li><a href="https://marketplace.visualstudio.com/items?itemName=mauve.terraform">Terraform</a></li>
</ol>
</li>
 	<li><a href="https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell">Powershell</a></li>
</ol>
<del>En terme de configuration j'ai ajouté les changements suivants dans le fichier settings.json :</del>
<pre><del>{
"workbench.statusBar.visible": true,
"editor.rulers": [80,100],
"editor.tabSize": 2,
"editor.insertSpaces": true,
"editor.detectIndentation": true,
"editor.wrappingColumn": 150,
"terraform.path": "C:\\Users\\edeneuve\\Desktop\\terraform.exe",
"extensions.autoUpdate": false,
"window.zoomLevel": 4
}</del></pre>
[Update 20/03/2017] Je viens de publier, dans mon repository GitHub, deux éléments :
<ol>
 	<li>arm-snippets.json :
Il s'agit de quelques snippet vscode a ajouter dans json.json via Ctrl+P (cmd sur mac :)) "&gt; Extrait de code utilisateurs" puis sélectionner JSON et coller le contenu du fichier dedans...
pour utiliser ces snippets, taper tout simplement "arm-ed" et vous aurez la liste :)</li>
 	<li>settings.json
Il s'agit tout simplement de mon fichier de settings de vscode, je le mets a jours souvent !</li>
</ol>
&nbsp;