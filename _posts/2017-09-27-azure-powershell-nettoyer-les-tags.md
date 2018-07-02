---
ID: 298
post_title: 'Azure Powershell &#8211; Nettoyer les Tags'
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2017/09/27/azure-powershell-nettoyer-les-tags/
published: true
post_date: 2017-09-27 10:48:30
---
Quand on fait de l'Azure, il peut arriver qu'on souhaite nettoyer les Tags, pour de multiples raisons.

Voici un peu  de Powershell pour :
<ol>
 	<li>Enlever les Tags des Ressources Groupes</li>
 	<li>Enlever les Tags des Ressources</li>
 	<li>et enfin Supprimer les Tags</li>
</ol>
Comme hier, dans notre script, il faut être connecter sur Azure donc on utilise la <a href="https://etienne.deneuve.xyz/2017/09/26/azurepscmdnotfound/">fonction d'hier</a> (pratique :)):
<pre><code class="PowerShell hljs"><span class="hljs-keyword">function</span> Check-AzureRMSession () {
<span class="hljs-variable">$Error</span>.Clear()
<span class="hljs-comment">#if context already exist</span>
<span class="hljs-keyword">try</span> {
Get-AzureRmVM -ErrorAction Stop | <span class="hljs-built_in">Out-Null</span>
}
<span class="hljs-keyword">catch</span> [System.Management.Automation.PSInvalidOperationException] {
Login-AzureRmAccount
}
<span class="hljs-variable">$Error</span>.Clear();
}</code></pre>
Ensuite la fonction :
<pre><code class="PowerShell hljs">function Remove-AzureRMAllTags () {
Get-AzureRmResourceGroup | Out-GridView -PassThru | Set-AzureRmResourceGroup -Tag @{}
Get-AzureRmResource | Select Name,ResourceType,Tags,ResourceGroupName |  Out-GridView -PassThru | Set-AzureRmResource -Tag @{} -Force
Get-AzureRMTag |  Out-GridView -PassThru | Remove-AzureRMTag 
}
</code></pre>
<code class="PowerShell hljs">Out-GridView -PassThru</code> Vous permet de choisir ceux que vous souhaitez supprimer :)