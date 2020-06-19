---
ID: 316
title: 'Microsoft Expérience 17 &#8211; VSTS For OPS part 1 !'
author: etienne.deneuve
post_excerpt: ""
layout: layouts/post-sidebar.njk
mySlug: vsts-for-ops-1
permalink: "{{ page.date | date: '%Y/%m/%d' }}/{{ mySlug }}/index.html"

published: true
date: 2017-10-09 18:30:55
---
Dans cet article, je vous détaille la partie (cachée) de ma démo lors de ma session au Microsoft Expérience 17 avec Stanislas Quastana. Le but de cet article est de préparer le terrain pour les articles suivants. Les prochains arriverons rapidement, avec dans l'idée, de vous aider à mieux appréhendez le CI/CD en tant qu'OPS, pour des sujets qui nous concernent, l'infra as code.
Voici le chemin que nous allons suivre :

1. Préparation de l'environnement [nous sommes ici]
2. Préparation d'une image de base Linux
3. Préparation d'une image de base Windows
4. Utilisation des images de base pour les spécialiser, afin de les rendre "Immutables"
5. Déploiement d'image en CI/CD avec Packer et Terraform depuis VSTS
Nous utiliserons des technologies Microsoft (VSTS, Azure, Windows Server...) mais aussi HashiCorp (Packer, Terraform) ainsi que des technologies Open Source (Linux..). Il n'est normalement pas nécessaire d’être, ni un maître du cloud, ni un demi dieu de l'infra as code pour suivre cette mini séries de 4 articles.

# Création des comptes nécessaire

Il faut créer un compte sur Visual Studio Team Services [ici](https://go.microsoft.com/fwlink/?LinkId=307137) et un compte Azure [ici](https://azure.microsoft.com/fr-fr/offers/ms-azr-0044p/).

# Création du Workspace VSTS

## Création du Projet
<https://{votrenomavous}.visualstudio.com/_projects?_a=new>
Indiquez le nom que vous souhaitez et sélectionnez Git comme "Version Control" :

<img class="alignnone wp-image-322 size-full" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/vsts-1-1.png" alt="" width="1680" height="793" />

## Initialisation du repository

<img class="alignnone size-full wp-image-323" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/vsts-2-1.png" alt="" width="1675" height="697" />
## C'est prêt !
<img class="alignnone size-full wp-image-324" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/vsts-3-1.png" alt="" width="1239" height="326" />

Voila, 3 étapes, notre premier projet est "prêt".

# Préparation de l'Agent pour VSTS

## Préparation de la VM

Dans Azure, avec le [template suivant](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-linux), déployez une vm Ubuntu basique.
Installez les prérequis de l'agent VSTS via les commandes suivantes :

```
sudo apt-get install -y libunwind8 libcurl3
sudo apt-add-repository ppa:git-core/ppa
sudo apt-get update
sudo apt-get install git
apt-get install libcurl4-openssl-dev
```

<h2>Installation de l'agent</h2>
<h3>Préparation dans VSTS</h3>
Accédez à la partie sécurité https://{votrenomavous}.visualstudio.com/_details/security/tokens et créer un PAT (Personnal Access Tokens).
<blockquote>Attention, c'est comme un mot de passe, il ne faut pas le diffuser. (Keepass, LastPass et autres peuvent les stocker correctement!).</blockquote>
Dans notre cas, l'agent devra avoir les droits Agent Pools et Deployment Group, car nous n'utiliserons qu'un seul agent. En prod, séparez-les !
<h3>Installation de l'agent</h3>
```
wget https://github.com/Microsoft/vsts-agent/releases/download/v2.123.0/vsts-agent-ubuntu.16.04-x64-2.123.0.tar.gz -O vsts-agent-ubuntu.16.04-x64-2.123.0.tar.gz
mkdir agent &amp;&amp; cd agent
tar -xvf ../vsts-agent-ubuntu.16.04-x64-2.123.0.tar.gz
./config.sh

# Répondez aux questions
sudo ./svc.sh install
sudo ./svc.sh start

```
Pour plus d'info sur l'[agent VSTS](https://docs.microsoft.com/fr-fr/vsts/build-release/actions/agents/v2-linux)
<h3>Vérification dans VSTS</h3>
Si vous accédez à l'url https://{votrenomavous}.visualstudio.com/_admin/_AgentPool vous devriez voir que votre agent est en vert !
<img class="alignnone size-full wp-image-321" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/Capture-Agent-Pool.png" alt="" width="565" height="145" />
