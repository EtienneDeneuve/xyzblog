---
ID: 352
title: >
  Microsoft Expérience 17 – VSTS For
  OPS part 3 !
author: etienne.deneuve
post_excerpt: ""
layout: layouts/post-sidebar.njk
mySlug: vsts-for-ops-part-3
permalink: "{{ page.date | date: '%Y/%m/%d' }}/{{ mySlug }}/index.html"

published: true
date: 2017-10-25 10:11:10
---
Dans cet article, je vous détaille la partie (cachée) de ma démo lors de ma session au Microsoft Expérience 17 avec Stanislas Quastana. Le but de cet article est de créer notre première build ! Les prochains arriveront rapidement, avec dans l’idée, de vous aider à mieux appréhender le CI/CD en tant qu’OPS, pour des sujets qui nous concernent, l’infra as code.
Voici le chemin que nous allons suivre :
1. Préparation de l’environnement
2. Préparation d’une image de base Linux
3. Préparation d’une image de base Windows [nous sommes ici]
4. Utilisation des images de base pour les spécialiser, afin de les rendre “Immutables”
5. Déploiement d’image en CI/CD avec Packer et Terraform depuis VSTS
Nous utiliserons des technologies Microsoft (VSTS, Azure, Windows Server…) mais aussi HashiCorp (Packer, Terraform) ainsi que des technologies Open Source (Linux..). Il n’est normalement pas nécessaire d’être, ni un maître du cloud, ni un demi dieu de l’infra as code pour suivre cette mini série de 4 articles.

# Introduction

Voici un petit récapitulatif ce qu'on aimerait :

<img class="alignnone size-full wp-image-365" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/Packer.png" alt="" width="1035" height="472" />

Vous devez créer deux comptes de stockage dans Azure vous même, un de "Production" et un "Eval" vous pouvez le faire avec Azure Cloud Shell depuis le portail Azure :

```
#create resource group
az group create -l westeurope -n vstsforops
#create storage account
az storage account create -n sto0eval -g vstsforops -l westeurope --sku Standard_LRS
az storage account create -n sto0prod -g vstsforops -lwesteurope --sku Standard_LRS
```

Du coup je vous ai déjà préparer les éléments histoire de ne pas perdre de temps sur la construction des templates packer. Récupérez l'archive ici : <a href="https://github.com/EtienneDeneuve/vsts-for-ops/archive/master.zip">   Download ZIP</a>, puis copiez les fichiers dans votre répertoire que vous avez cloné la fois dernière. J'ai fais quelques modifications sur la hiérarchie des dossiers afin que ça soit plus évolutif dans le temps, nous y reviendrons plus tard.

Le fichier packer qui nous intéresse est dans le dossier /storage/windows/base/windows_base_packer.json, il est tout a fait standard, comme ceux que Stanislas détaille ici : https://github.com/squasta/PackerAzureRM

Vous noterez peut être que j'ai ajouté un provisionner "windows-update" :

```json

&quot;provisioners&quot;: [
{
&quot;type&quot;: &quot;windows-update&quot;
},

```

Celui n'est pas standard, vous devez le mettre dans le même répertoire que votre exécutable packer, il est téléchargeable à l'adresse suivante : https://github.com/rgl/packer-provisioner-windows-update/releases mais je l'ai aussi inclus dans mon repository git au passage nous allons également mettre la dernière version de Packer. Ces actions sont à faire sur la machine "Agent" de <a href="https://etienne.deneuve.xyz/2017/10/09/vsts-for-ops-1/">vsts-for-ops-1</a> :

```
apt install unzip
wget https://github.com/rgl/packer-provisioner-windows-update/releases/download/v0.4.0/packer-provisioner-windows-update-linux.tgz
wget https://releases.hashicorp.com/packer/1.1.0/packer_1.1.0_linux_amd64.zip
tar -xvf ./packer-provisioner-windows-update-linux.tgz
unzip packer_1.1.0_linux_amd64.zip
mv ./packer-provisioner-windows-update-windows /usr/local/bin/
mv ./packer /usr/local/bin/
```
Si vous n'avez pas installé azure cli et l'agent, vous pouvez aussi utiliser ce script sur votre vm agent:

https://raw.githubusercontent.com/EtienneDeneuve/vsts-for-ops/master/scripts/platform/agent/prepare_vm.sh

# Bon et La build ?

Ouvrez VSTS :

puis cliquez sur "build &amp; release" :

<img class="alignnone size-full wp-image-372" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/Sans-titre.png" alt="" width="592" height="164" />

Puis sur "+ New"

<img class="alignnone size-full wp-image-361" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/vsts-create-1.png" alt="" width="1027" height="158" />

Puis sur "Empty process"

<img class="alignnone size-full wp-image-362" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/vsts-create-2.png" alt="" width="1035" height="185" />

Puis complétez le champs "Name" avec par exemple "Packer-Base-Windows 2012 R2"

<img class="alignnone size-full wp-image-363" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/vsts-create-3.png" alt="" width="1035" height="475" />

Dans la Phase 1 nous allons ajouter deux étape "build immutable image" et "azure cli" :

<img class="alignnone size-full wp-image-353" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/vsts-create-4.png" alt="" width="1035" height="504" />

Renseignez les variables telles que ci dessous :

Packer Template : "User Provided"

Packer template location, naviguer vers "storage/Windows/base/windows_base_packer.json"

Dans le template parameters (je suis sympa, copiez collez ca :):

```json

{

&quot;client_id&quot;:&quot;$(ARM_CLIENT_ID)&quot;,

&quot;client_secret&quot;:&quot;$(ARM_CLIENT_SECRET)&quot;,

&quot;resource_group_name&quot;:&quot;$(ARM_RESOURCE_GROUP_NAME)&quot;,

&quot;storage_account&quot;:&quot;$(ARM_STORAGE_ACCOUNT)&quot;,

&quot;subscription_id&quot;:&quot;$(ARM_SUBSCRIPTION_ID)&quot;,

&quot;object_id&quot;:&quot;$(ARM_OBJECT_ID)&quot;,

&quot;tenant_id&quot;:&quot;$(ARM_TENANT_ID)&quot;,

&quot;windows_sku&quot;:&quot;$(WINDOWS_SKU)&quot;,

&quot;build_resquestfor&quot;:&quot;$(BUILD_REQUESTEDFOR)&quot;,

&quot;location&quot;:&quot;$(ARM_LOCATION)&quot;,

&quot;vm_size&quot;:&quot;$(ARM_VM_SIZE)&quot;,

&quot;capture_name&quot;:&quot;$(CAPTURE_NAME)&quot;

}

```

Puis, dans "Output", renseigner IMAGEURI :

<img class="alignnone size-full wp-image-354" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/vsts-create-5.png" alt="" width="1108" height="727" />

Ajoutez maintenant une tâche "Azure CLI" :

<img class="alignnone size-full wp-image-355" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/vsts-create-6.png" alt="" width="1035" height="560" />

Sélectionnez votre souscription azure (si elle n'est pas dans la liste, cliquez sur manage et ajoutez la) et indiquez le chemin vers le script bash "script/platform/tasks/az-move-vhd.vhd" et ajoutez les arguments "-d "$(DESTACCOUNTKEY)" -s "$(SOURCEACCOUNTKEY)""

<img class="alignnone size-full wp-image-356" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/vsts-create-7.png" alt="" width="1109" height="648" />

Une fois que ces deux tâches sont complétées, nous allons déclarer nos variables (si vous ne savez pas comment les récupérées, suivez le guide sur le dépôt git de Stanislas plus haut) :

<img class="alignnone size-full wp-image-357" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/vsts-create-8.png" alt="" width="1112" height="893" />

Vous remarquerez que certaines sont masquées, rien de bien compliqué, cliquez sur le cadenas et elle sera "protégée" :

<img class="alignnone size-full wp-image-358" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/vsts-create-9.png" alt="" width="990" height="108" />

Dans l’idéal, les variables a protégées seraient : ARM_CLIENT_SECRET, ARM_SUBSCRIPTION_ID, ARM_TENANT_ID, DESTACCOUNTKEY, SOURCEACCOUNTKEY mais pour des questions de simplifications dans cet exercices, nous ne protégerons que les clés du compte de stockage, nous corrigerons par la suite.

Pour récupérer les clés des comptes de stockage sélectionnez votre compte de stockage dans le resource group que vous avez créer et allez dans "Access Keys" :

<img class="alignnone size-full wp-image-360" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/storage-account-key.png" alt="" width="1108" height="557" />

Ensuite, comme nous allons construire des images Packer Windows, il y a des chances pour que la build dure plus d'une heure (Mise à jour etc) donc dans les options de la build nous allons passer le time out de la build a 240 minutes :

<img class="alignnone wp-image-378 size-full" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/10/timeout-e1508918816983.png" alt="" width="1225" height="545" />

Cliquez maintenant sur "Save &amp; Queue".

&nbsp;

Votre build est maintenant lancé.

Dans le prochain article nous verrons en détail cette partie de "build" et surtout comprendre comment ça fonctionne (ou pas :))

&nbsp;

&nbsp;