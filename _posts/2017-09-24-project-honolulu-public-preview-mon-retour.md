---
ID: 225
post_title: '[update] Project Honolulu Public Preview &#8211; Mon retour'
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2017/09/24/project-honolulu-public-preview-mon-retour/
published: true
post_date: 2017-09-24 19:12:41
---
<h1>Intro</h1>
Au cas ou vous auriez raté l'annonce concernant le projet Honolulu qui vise à remplacer les MMC de nos servers Windows, je vous invite à regarder les quelques liens ci-dessous pour en savoir plus :
<ul>
 	<li>https://seyfallah-it.blogspot.fr/2017/09/honolulu-project.html</li>
 	<li>aka.ms/honoluludownload</li>
 	<li>https://blogs.technet.microsoft.com/askpfeplat/2017/09/20/project-honolulu-a-new-windows-server-management-experience-for-the-software-defined-datacenter-part-1/</li>
 	<li>https://gotoguy.blog/2017/09/24/secure-access-to-project-honolulu-with-azure-ad-app-proxy-and-conditional-access/</li>
</ul>
Pour ma part, voici mon retour sur cette nouveauté, d'un prime abord, c'est excellent, un simple MSI à installer et on peut commencer à ajouter ses serveurs dans l'interface. Génial ça fonctionne depuis mon téléphone, depuis mon Mac... Bref, c'est génial!
<h1>Quelques trucs cools :</h1>
<ul>
 	<li>l'affichage des VMs :</li>
</ul>
<img class="alignnone wp-image-226" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/09/vm.png" alt="" width="1541" height="249" />
<ul>
 	<li>ASR (Azure Site Recovery) Automatisé :</li>
</ul>
<img class="alignnone wp-image-227" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/09/Honolu.png" alt="" width="1710" height="922" />

(Bon, y en a pleins d'autres, je vous laisse des surprises #nospoil)
<h1>Les trucs que j'aurais aimés :</h1>
<ul>
 	<li>Dans l'installer proposer directement un certificat let's encrypt plutôt qu'un auto-signé (2017!)</li>
 	<li>Au niveau de l'import :</li>
 	<li>
<ul>
 	<li>L'import des servers depuis l'AD et le DNS (à la 2012)</li>
 	<li>L'import des VM (Windows) existantes dans Azure</li>
 	<li>Ajout des machines virtuelles lors de l'ajout de serveurs avec installé Hyper V</li>
</ul>
</li>
 	<li>La création d'une vault pour stocker ses "Credentials" (à la cmdkey /add), pour sélectionner ensuite le compte que l'on souhaite.</li>
 	<li>L'absence de centralisation (à la 2012), il aurait été cool d'avoir une overview complète comme celle-ci.</li>
 	<li>L'absence de Powershell Launcher, ça aurait été cool de pouvoir lancer des commandes Powershell depuis cette interface.</li>
 	<li>Pas de containers, j'aime bien les containers moi, j'aurais aimé trouver une petite interface comme Cockpit sur Fedora : <img src="https://bobcares.com/wp-content/uploads/2015/08/docker-management-ui.png" /></li>
</ul>
Voici les éléments pour les quels j'ai voté sur le User Voice Honolulu : https://windowsserver.uservoice.com/users/674735389-etienne-deneuve
<h1>Ma petite conclusion:</h1>
Et vous ? Vous l'avez essayé déjà ? Qu'en pensez-vous ? La direction est bonne, un moteur d'extensions (https://github.com/hongtao-chen/hello-honolulu) est déjà prévu, et presque en place... On va pouvoir arrêter de mettre des GUI sur tous les serveurs ?