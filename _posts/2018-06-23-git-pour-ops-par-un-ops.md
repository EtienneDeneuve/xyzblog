---
ID: 422
post_title: Git pour Ops, par un Ops
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2018/06/23/git-pour-ops-par-un-ops/
published: true
post_date: 2018-06-23 19:57:39
---
Git est un logiciel de contrôle de code source déjà très utilisé dans le monde des dev. Lorsque l'on fait de l'Infra as Code, nous pouvons aussi utiliser git, je pense même que cela va devenir une vraie nécessité. Pour nous les Ops, ce n'est pas nécessairement naturel et j'ai donc décidé de démystifier git dans un contexte Cloud comme Azure.

Afin de suivre ce billet, vous devez installer quelques outils sur votre poste de travail afin de pouvoir tester par vous-même.

Je vous invite également à lire ces articles :
- [Git for ops, le retour](https://etienne.deneuve.xyz/2018/06/28/git-pour-les-ops-par-un-ops-le-retour/)
- [Mon Setup de VS Code pour Bash et Git](https://etienne.deneuve.xyz/2018/06/26/setup-vs-code-bash-git/)
- [Terraform](https://etienne.deneuve.xyz/2017/10/01/microsoft-experience-17-infrastructure-code-modelisez-et-provisionnez-vos-services-azure-avec-terraform-et-packer)
- [Visual Studio Code](https://etienne.deneuve.xyz/2017/10/09/vsts-for-ops-1/)

## Sur le poste de travail

Sur un poste Windows vous devez installer à minima :

- Visual Studio Code
- Git-SCM

Vous pouvez utiliser chocolatey :
<script src="https://gist.github.com/EtienneDeneuve/5738b4f0aacac785c2a7f982f0346f5d.js"></script>

Sur macOs :

- Visual Studio Code
- git

## Un serveur Git

Vous pouvez utiliser [VSTS](https://go.microsoft.com/fwlink/?LinkId=307137) ou [GitHub](https://github.com/).

### Repo sur GitHub

Une fois votre compte actif, créez un repository Git via le bouton vert "Create Repository"

### Repo sur Vsts

J'ai déjà publié un article concernant vsts<a href="https://etienne.deneuve.xyz/2017/10/09/vsts-for-ops-1/"> ici</a>

### Git en SSH

Afin d'avoir accès à VSTS ou à GitHub depuis un bash Windows ou Linux, il faut générer une clé SSH :
> Je vous recommande vivement l'utilisation de bash pour git et la suite de cet article. Si vous n'avez pas encore installer bash sur votre Windows, installer le puis suivez cet article : <a href="https://etienne.deneuve.xyz/2018/06/26/setup-vs-code-bash-git/" target="_blank" rel="noopener">Open Source + Windows = Vs Code + Bash + Git</a>

```bash
mkdir -p ~/.ssh
ssh-keygen -f ~/.ssh/vsts
cat vsts.pub
```

Ouvrez VSTS ou Git puis :

VSTS : https://&lt;votreurl&gt;/_details/security/keys)&lt;/votreurl&gt;Git : https://github.com/settings/keys

Cliquez sur &quot;Add&quot; et collez la clé publique.

Ensuite ajoutez la clé dans l&#039;agent ssh :

```bash
ssh-agent bash
ssh-add ~/.ssh/vsts
```

# Where to start ?

Lorsqu&#039;on commence à travailler avec git, il est possible d&#039;utiliser deux méthodes, soit on clone le repo distant, soit on ajoute le repo distant comme source.

## Clone ou Remote ?

Pour faire un clone ou ajouter un repository distant, il suffit de récupérer l&#039;url disponible dans l&#039;interface de GitHub ou de VSTS puis avec cette url :

- Clone

```bash
git clone https://xxxx/git.git
# ou avec la belle cle ssh :
git clone ssh://
```

&gt; Le clone est utilisable lorsque notre projet est vide, ou qu&#039;il n&#039;est pas encore present dans la machine de travail (reinstallation, pc fixe de la maison...)

- Remote

Cette méthode est plus complexe et pas forcément nécessaire, il est juste intéressant de la connaitre.

```bash
git remote add origin url
git commit . -m "inital commit"
git push origin master
git add --all
git commit . -m "add exisiting file in the remote repo"
git push origin master
```

> Le remote est utilisable si le projet est déjà sur la machine et que le repository distant n'existe pas encore.
> Je vous conseille de bien faire un premier commit "vide" puis d'ajouter les fichiers (git add --all) puis à nouveau d'envoyer les fichiers.

## Ajouter/Supprimer des fichiers à l'index

Avec Git, quand on créer un nouveau fichier, par défaut il n'est pas forcément ajouté à l'index, c'est à dire qu'il ne sera pas envoyé au server lors d'un ``git push``.

- Ajout de fichiers

```bash
git add ./monchemin/monfichier
# ou un dossier complet :
git add ./mondossier/
```

> Attention, git ne prendra jamais un dossier vide, il ne le prendra que si un fichier meme vide est present.

- Suppression de fichiers

> *Je ne suis pas responsable de vos fichiers perdus ;)*
> Lorsqu'on supprime un fichier du repertoire local, il n'est pas supprimé de l'index de git sans faire la commande :

```bash
git rm ./monchemin/monfichier --cached
```

## Commit

Afin d'ajouter nos fichier dans l'index en attente, il est nécessaire de valider ces fichiers, un peu comme en base de données Commit, Execute, Rollback, en cas de soucis.

```bash
git commit -m &quot;le message du commit en fonction de votre travail... (pensez à vos collègues !&quot;
```

## Pull ou Fetch ?

Avant de valider ce ``commit`` sur le server, il est important de vérifier si on est bien raccord avec le canal "upstream", c'est à dire le serveur distant.

- Pull

```bash
git pull
```

> Git pull en réalité, c'est un ``git fetch`` puis un ``git merge FETCH_HEAD``, pour gagner du temps.

- Fetch

```bash
git fetch
```

> Git fetch permet de récupérer les commits distants qui on eu lieu de puis le dernier ``fetch``. En effet, sans faire de ``pull`` ou de ``fetch``, le server distant et local on des versions différentes des index (et donc des fichiers).

## Git push

Afin de partager votre travail avec vos collègues, vous devez envoyer vos modifications sur le server (si, si, si).
Il y a plusieurs façons de travailler avec git, mais lorsqu'on est dans une branche de travail (et pas de prod!) nous pouvons faire un push :

```bash
git push
```

## Git branch

Avec git, l'idéal c'est de créer des branches pour le travail et pour la production.
J'ai beaucoup aimer la façon de gérer les branches que j'ai utilisé avec [Yannick Neff](https://www.linkedin.com/in/yannick-neff-7754aa8/) et [Maxence Maireaux](https://www.linkedin.com/in/maxencemaireaux/) sur l'[Os Factory Open Source by Société Générale](github.com/societe-general/os-factory).
En gros, on part du postulat qu'il y aura plusieurs versions du produit et donc dès le départ, on créer une branche v0 pour la version initial et on n'utilise pas la branch master (du tout). la v0 sera la version initiale de production du produit, en suite on créer des branches par feature request.

- la premiere branche :

```bash
git checkout -b v0
```

- puis les autres :

```bash
# support d&#039;aws
git checkout -b v0-aws
# support d&#039;azure
git checkout -b v0-azure

```

Voilà, vous savez maintenant utiliser les bases de git, je vous invite à poursuivre avec <a href="https://etienne.deneuve.xyz/2018/06/26/setup-vs-code-bash-git/" target="_blank" rel="noopener">Open Source + Windows = VS Code + Bash + Git</a>.