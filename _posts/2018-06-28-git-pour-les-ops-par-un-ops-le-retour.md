---
ID: 452
post_title: Git pour les Ops, par un ops le retour
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2018/06/28/git-pour-les-ops-par-un-ops-le-retour/
published: true
post_date: 2018-06-28 13:43:00
---
Dans le premier article [git for ops](https://etienne.deneuve.xyz/2018/06/23/git-pour-ops-par-un-ops/) nous avons appris à :

- Créer un repo sur Github ou VSTS
- Configurer une clé ssh pour Git
- Ajouter/Supprimer des fichiers à l'index local
- Commit des fichiers dans l'index local
- Cloner un repo ou ajouter une source distante
- Récupérer le travail existant dans le repo distant
- Envoyer ces commits dans le repository distant
- Créer une branche

<!--more-->

Dans cet article, nous allons aller un peu plus loin et apprendre quelques éléments de git qui sont primordiaux lorsque l'on travaille avec des branches.

## Git checkout

La commande git checkout permet de se changer de branche de travail. C'est à dire que si nous faisons la commande ``git checkout masuperfeature``, les fichiers et les dossiers seront ceux de la branche en question. Faisons donc un petit exercice pour bien comprendre comment cela fonctionne :

&gt; Vous pouvez créer un repository git local sans server pour cet exercice, c'est que j'ai fais moi même et par conséquent je n'utilise pas les commande ``git push`` et ``git pull`` pour simplifier cet exercice.

```bash
# on se place dans la branche master
git checkout master
# on creer un fichier vide :
touch monfichier.txt
# on l&#039;ajoute au &quot;stage&quot; :
git add monfichier.txt
# puis on commit :
git commit -m &quot;mon commit dans master&quot;
# puis on change de branch, le -b permet de créer la branche si elle n&#039;existe pas encore
git checkout -b manouvellebranche
# on valide qu&#039;elle est bien a jour par rapport au serveur
ls
# qui nous retourne :
monfichier.txt
```

Maintenant, ajoutons un fichier dans cette nouvelle branche :

```bash
touch monfichier2.txt
git add monfichier2.txt &amp;&amp; git commit -m &quot;mon commit dans manouvellebranche&quot;
```

Retournons dans la branche "master" :

```bash
# on change vers master
git checkout master
# on creer un nouveau fichier :
touch monfichiermaster.txt
git add monfichiermaster.txt &amp;&amp; git commit -m &quot;ajout du fichier dans master&quot;
ls
# nous retourne :
monfichier.txt monfichiermaster.txt
# si nous retournons dans la branche manouvellebranche :
git checkout manouvellebranche
ls
monfichier2.txt monfichier.txt
```

Conclusion: les branches permettent d'isoler le travail sans se mélanger entre collègues ou entre features.

## Git rebase

Il est assez fréquent de travailler sur plusieurs branches en parallèle et qui nous oblige lors de modification importante dans la branche principale de faire "redescendre" les modifications de la branche "master" (ou celle qui a servi de base à la création). Nous allons voir comment récupérer ces modifications dans notre branche "manouvellebranche" :

```bash
git checkout manouvellebranch
ls
git rebase master
git checkout manouvellebranch
First, rewinding head to replay your work on top of it...
Applying: mon commit dans manouvellebranche
```

## Git Merge

Une fois que notre feature est prête, il peut être intéressant de migrer le travail dans la branche "mère". Dès lors nous allons pouvoir "merger", il existe plusieurs façon de merger, soit on merge tout avec l'historique de chaque commit dans la branche "mère", soit on merge sans l'historique, rassembler dans un seul commit, je préfère cette option afin de ne pas polluer l'historique de la branche de "production" :

```bash
# ajoutons quelques fichiers dans la branche manouvellebranche
touch toto{0..9}{0..9}.txt
git add toto*
git commit -m &quot;adding 100 files&quot;
```

Ensuite, changeons de branche puis effectuons un merge, simple (le moins bien):

```bash
git checkout master
git merge manouvellebranch
```

Et si je me suis trompé ?

```bash
# d&#039;abord on vérifie l&#039;id du commit : (-B permets de récupéré 6 lignes de avant le contexte, -A la meme chose mais apres le contexte)
git log | grep -B 6 -A 6 adding

commit 126d0fe758ebb8b7c155d58968063edec996a673
Author: Etienne Deneuve &lt;etienne.deneuve@cellenza.com&gt;
Date: Thu Jun 28 13:07:27 2018 +0200

adding 100 file

commit c9323cb10debb1a0a028f9e7db65a622fb8906a1
Author: Etienne Deneuve &lt;etienne.deneuve@cellenza.com&gt;
Date: Thu Jun 28 12:27:02 2018 +0200

mon commit dans manouvellebranche
# puis on annule notre commit en prenant celui juste avant notre erreur :
git reset c9323cb10debb1a0a028f9e7db65a622fb8906a1
git status
On branch master
nothing to commit, working tree clean
```

Cela tombe, bien nous allons pouvoir faire le deuxième merge (le mieux):

```bash
git checkout master
git merge --squash manouvellebranch
git status
On branch master
Changes to be committed:
(use &quot;git reset HEAD ...&quot; to unstage)

new file: toto00.txt

new file: toto99.txt
# dès lors, nous devons commiter:
git commit -m &quot;oui, c&#039;est bon on y va !&quot;
```