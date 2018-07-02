---
ID: 135
post_title: 'Azure et le backup ? (d&#8217;un Mac)'
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2017/03/14/azure-et-le-backup-dun-mac/
published: true
post_date: 2017-03-14 22:29:37
---
Un petit article "Quick &amp; very very dirty" qui est soyons honnêtes complètement inutile.

Mais quand c'est inutile, c'est possible que ce soit fun !

/!\ Disclaimer : Je suis pas responsable de vos données perdues ! /!\
<iframe class="giphy-embed" src="//giphy.com/embed/ULsH97rqqbf68?html5=true" width="480" height="269" frameborder="0" allowfullscreen="allowfullscreen"></iframe>

<a href="http://giphy.com/gifs/nuke-atomic-bomb-a-ULsH97rqqbf68">via GIPHY</a>
<h3>Pré-requis</h3>
Du crédit Azure, un Mac avec Sierra.
<!--more-->
<h3>Azure Side</h3>
<h4>Préparation sur un Mac :</h4>
Comme on est sur macOs, on va commencer par utiliser le CLI azure, dispo ici : <a href="http://aka.ms/mac-azure-cli">Programme d’installation Mac OS X</a> (aka.ms/mac-azure-cli). Je vous passe la bête installation Next Next Finish...

Une fois bien installé sur le macOs, ouvrez le terminal lancez les commandes suivantes :

[code]azure login[/code]

suivez le liens https://aka.ms/devicelogin avec le code généré dans le terminal

puis vérifiez vos informations de facturation avec :

[code]azure account show[/code]
<h4>Création de l'environnement Azure :</h4>
On crée un petit resource group qui va bien (c'est du Poc Quick &amp; very Dirty, mais on est pas des sauvages !)

[code]azure group create RG_Backup NorthEurope[/code]

et si tout se passe bien :
<pre>info:    Executing command group create
+ Getting resource group RG_Backup
+ Creating resource group RG_Backup
info:    Created resource group RG_Backup
data:    Id:                  /subscriptions/XXXXXXXXXXXX/resourceGroups/RG_Backup
data:    Name:                RG_Backup
data:    Location:            northeurope
data:    Provisioning State:  Succeeded
data:    Tags: null
data:
info:    group create command OK
</pre>
On va créer maintenant un compte de stockage :
<pre>azure storage account create stobackupmac \
--kind Storage \
--sku-name LRS \
--resource-group RG_BACKUP \
--location NorthEurope
</pre>
Normalement, si tout va bien on a un petit message "info: storage account create command OK", continuons, nous avons besoin d'une clé sur ce compte de stockage :
[code]azure storage account keys list stobackupmac -g RG_Backup[/code]
on récupère des clefs d'accès ici :
<pre>info:    Executing command storage account keys list
+ Getting storage account keys                                                 
data:    Name  Key                                                                                       Permissions
data:    ----  ----------------------------------------------------------------------------------------  -----------
data:    key1  ONsztQjA44F+S+ffVXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXTNROw1yPOQdcHVynXOOt3g==  Full       
data:    key2  dppNGuvyIn1li/ffVXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXTNROw1yPOQdcHVynXOOt3g==  Full       
info:    storage account keys list command OK
</pre>
Maintenant, on crée un partage Azure :
Le plus simple, c'est de créer des variables d'environnement pour se simplifier l'exécution des commandes qui vont suivre :
<pre>export AZURE_STORAGE_ACCOUNT=stobackupmac
export AZURE_STORAGE_ACCESS_KEY=dppNGuvyIn1li/ffVXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXTNROw1yPOQdcHVynXOOt3g==</pre>
Ainsi on peut maintenant taper la commande suivante :

[code]azure storage share create --share sharebkp --quota 100[/code]

Ce qui nous donnera "info:    storage share create command OK" et donc, c'est prêt !
<h3>Mac Side</h3>
Connectons le mac avec le Share dans Azure. Je suppose que le port TCP 445 est ouvert en sortie de chez vous.

Ouvrez le Finder puis Menu "Aller", "Se connecter au serveur..." (ou Command + K)

Indiquez : smb://sharebkp@stobackupmac.file.core.windows.net/sharebkp (si vous avez tout suivi)

au prompt du mot de passe, indiquez "sharebkp" et une des clés comme celle exportée dans "AZURE_STORAGE_ACCESS_KEY". Normalement, vous devriez le voir dans le Finder correctement.

On arrête la partie graphique du mac et on retourne dans un terminal :
<pre>cd /Volumes/sharebkp
#création du SparseBundle
hdiutil create \
-library NONE \
-fs HFS+ \
-volname "Azure Time Machine" \
-encryption AES-128 \ #Attention, oubliez pas le mot de passe !
-size 99g \
MacBookPro-Etienne.sparsebundle
</pre>
Maintenant, allez allumer la cafetière, lancer un détartrage, nettoyez la, faites un café, ca devrait être prêt...
Si tout est bien prêt, avec le Finder, double cliquer sur l'image SparseBundle, vous devriez la retrouver dans "/Volumes/Azure\ Time\ Machine/", retournons dans le terminal :
<pre>sudo tmutil setdestination /Volumes/Azure\ Time\ Machine/
</pre>
Et Hop, Time Machine sur Azure !
(oui, je sais ;))