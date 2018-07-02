---
ID: 291
post_title: 'FRPSUG &#8211; Powershell Saturday et contenu de la session (de l&#8217;utile, ou pas)'
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2017/09/26/frpsug-powershell-saturday-et-contenu-de-la-session-de-lutile-ou-pas/
published: true
post_date: 2017-09-26 22:19:54
---
Suite au premier French PowerShell Saturday du <a href="https://frpsug.github.io">FRPSUG</a> à Paris Le Samedi 16 septembre, dans les locaux de <a href="http://www.cellenza.com/fr/">Cellenza.</a> (la où je bosse, pour ceux qui ne le savent pas!)

Si vous ne connaissez pas encore FRPSUG (Quoi???)
<blockquote>Utiliser le channel #french sur <a href="https://powershell.slack.com/Slack">PowerShell.slack.com</a> (<a href="http://slack.poshcode.org/">S’inscrire</a>)

Allez sur le GitHub : <a href="https://github.com/FrPSUG">https://github.com/FrPSUG</a></blockquote>
# Ma session :

Avec <a href="https://twitter.com/LopesMick">Mickael Lopes</a> on avait décidé de faire une session "à la cool" où on a montré quelques petits scripts inutiles (ou pas...)

Nous considérons que on peut tout aussi bien apprendre en s'amusant, l'event était un samedi, donc il fallait bien s'amuser un peu !

Je vous joins nos slides ! <a href="https://etienne.deneuve.xyz/wp-content/uploads/2017/09/FRPSUG.pptx">FRPSUG</a> (OpenSource MIT!)

# Première démo

Devant la foule en délire (ou pas... on s'est rendu compte que sur les pc type Surface, ça ne marche pas!) :

```Powershell
&lt;div&gt;start-job {&lt;/div&gt;
&lt;div&gt;[console]::beep(440,500)&lt;/div&gt;
&lt;div&gt;[console]::beep(440,500)&lt;/div&gt;
&lt;div&gt;[console]::beep(440,500)&lt;/div&gt;
&lt;div&gt;[console]::beep(349,350)&lt;/div&gt;
&lt;div&gt;[console]::beep(523,150)&lt;/div&gt;
&lt;div&gt;[console]::beep(440,500)&lt;/div&gt;
&lt;div&gt;[console]::beep(349,350)&lt;/div&gt;
&lt;div&gt;[console]::beep(523,150)&lt;/div&gt;
&lt;div&gt;[console]::beep(440,1000)&lt;/div&gt;
&lt;div&gt;[console]::beep(659,500)&lt;/div&gt;
&lt;div&gt;[console]::beep(659,500)&lt;/div&gt;
&lt;div&gt;[console]::beep(659,500)&lt;/div&gt;
&lt;div&gt;[console]::beep(698,350)&lt;/div&gt;
&lt;div&gt;[console]::beep(523,150)&lt;/div&gt;
&lt;div&gt;[console]::beep(415,500)&lt;/div&gt;
&lt;div&gt;[console]::beep(349,350)&lt;/div&gt;
&lt;div&gt;[console]::beep(523,150)&lt;/div&gt;
&lt;div&gt;[console]::beep(440,1000)&lt;/div&gt;
&lt;div&gt;}&lt;/div&gt;
&lt;div&gt;```</div>
<div>Ok bon, pour ceux qui ne peuvent reconnaître, c'est la marche impériale.</div>
<div></div>
<div># Deuxième Démo :</div>
<div>Comment on attaque les API de la SNCF en Powershell :</div>
<div><a href="https://etienne.deneuve.xyz/2015/12/16/get-nexttrain/">https://etienne.deneuve.xyz/2015/12/16/get-nexttrain/</a></div>
<div></div>
<div># Troisième Démo:</div>
<div>Comment on récupère le statut de l'encre sur un printer HP :</div>
<div><a href="https://etienne.deneuve.xyz/2016/01/15/get-ink-level-from-hp-printers-in-powershell/">https://etienne.deneuve.xyz/2016/01/15/get-ink-level-from-hp-printers-in-powershell/</a></div>
<div></div>
<div># Quatrième Démo :</div>
<div>Comment on se connecte à des PaloAlto en PowerShell !</div>
<div><a href="https://github.com/EtienneDeneuve/Powershell/blob/master/PaloAlto/Get-PaloAlto.ps1">https://github.com/EtienneDeneuve/Powershell/blob/master/PaloAlto/Get-PaloAlto.ps1</a></div>
<div></div>
<div># Cinquième Démo :</div>
<div>Manipulation de OneDrive via les API, en Powershell :</div>
<div><a href="https://github.com/EtienneDeneuve/Powershell/blob/master/OneDriveAPI/Remove-AllOneDriveSharedLink.ps1">https://github.com/EtienneDeneuve/Powershell/blob/master/OneDriveAPI/Remove-AllOneDriveSharedLink.ps1</a></div>
<div></div>
<div># Sixième Démo :</div>
<div>Un script que j'ai fait pour un client (merci !), pour mettre en place IpSec en mode transports sur les 5 ou 6 domaines :</div>
<div><a href="https://github.com/EtienneDeneuve/Powershell/blob/master/IpSec/Invoke-IPSec.ps1">https://github.com/EtienneDeneuve/Powershell/blob/master/IpSec/Invoke-IPSec.ps1</a></div>
<div></div>
<div>Vous pouvez bien sûr les utiliser, les modifier et m'en proposer d'autres !</div>
<div></div>
<div>Un grand merci à :</div>
<ul>
 	<li>l'équipe de Cellenza pour les locaux</li>
 	<li>Metsys pour la bouffe</li>
 	<li>FRPSUG pour être venu</li>
 	<li>à tous ceux qui sont venus</li>
 	<li>et également à ceux qui ne sont pas venus, la nourriture prévue pour eux est partie nourrir des personnes dans le besoin (Merci <a href="https://fr.linkedin.com/in/guillaume-mathieu-785431119">Guillaume Matthieu</a> !)</li>
</ul>
&nbsp;

Si vous étiez là, ça vous a plu ?
<div></div>