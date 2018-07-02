---
ID: 170
post_title: 'Azure Networking – TP – Part 2 &#8211; vNet Peering'
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2017/04/27/azure-networking-tp-part-2/
published: true
post_date: 2017-04-27 12:00:06
---
<a href="https://www.linkedin.com/in/lopesmickael">Mickael Lopes</a> et moi avons animé une session au Global Azure Bootcamp intitulée : "Le réseau dans Azure : Cas d'usage et Retour d'expériences".
Les slides de  notre session sont dispo sur slideshare ici : <a href="https://www.slideshare.net/MickaelLOPES91/gab-le-rseau-dans-azure">Slideshare de Mickael</a>
Comme prévu, voici des TP pour aller avec notre présentation, a faire vous même :)
Ce post est la suite de la partie 1, qui part du principe que vous avez déjà la partie 1 de faite sur votre tenant, dans toutes les versions (Powershell, Azure Cli 2.0 et ARM Template).
<h1>vNet Peering</h1>
L'objectif est de connecter nos 3 Vnets, avec nos 3 outils :
<ul>
 	<li>vnet-test-ps</li>
 	<li>vnet-test-az</li>
 	<li>vnet-test-json</li>
</ul>
<h2>Powershell</h2>
Nous allons créer le lien entre vnet-test-ps et vnet-test-az :
<pre><code class="lang-powershell"><span class="hljs-variable">$vnetPS</span> = <span class="hljs-pscommand">Get-AzureRmVirtualNetwork</span><span class="hljs-parameter"> -ResourceGroupName RG-Vnet-Exo-PS<span class="hljs-parameter"> -Name </span>vnet-test-ps</span>
<span class="hljs-variable">$vnetAZ</span> = <span class="hljs-pscommand">Get-AzureRmVirtualNetwork</span><span class="hljs-parameter"> -ResourceGroupName RG-Vnet-Exo-AZ -Name </span>vnet-test-az</code></pre>
Puis nous créons le lien PS2AZ :
<pre><code class="lang-powershell"><span class="hljs-pscommand">Add-AzureRmVirtualNetworkPeering</span><span class="hljs-parameter"> -Name </span>PS2AZ<span class="hljs-parameter"> -VirtualNetwork </span><span class="hljs-variable">$vnetPS</span><span class="hljs-parameter"> -RemoteVirtualNetworkId </span><span class="hljs-variable">$vnetAZ</span>.Id
</code></pre>
<pre><code class="lang-powershell"><span class="hljs-pscommand">Add-AzureRmVirtualNetworkPeering</span><span class="hljs-parameter"> -Name </span>AZ2PS<span class="hljs-parameter"> -VirtualNetwork </span><span class="hljs-variable">$vnetAZ</span><span class="hljs-parameter"> -RemoteVirtualNetworkId </span><span class="hljs-variable">$vnetPS</span>.Id</code></pre>
nous pouvons vérifié si le peering est fonctionnel via la commande Powershell :
<pre><code class="lang-powershell"> <span class="hljs-pscommand">Get-AzureRmVirtualNetworkPeering</span><span class="hljs-parameter"> -VirtualNetworkName </span>vnetPS<span class="hljs-parameter"> -ResourceGroupName </span>vnetPS<span class="hljs-parameter"> -Name </span>PS2AZ</code></pre>
en l'état, notre peering est configuré mais certaines options ne sont pas activées :
<ul>
 	<li>Forwarded Traffic</li>
 	<li>Gateway Transit</li>
 	<li>Remote Gateways</li>
</ul>
Dans notre cas, nous avons besoin d'activer le transfert du trafic.
<pre><code class="lang-powershell">$ps2az = <span class="hljs-pscommand">Get-AzureRmVirtualNetworkPeering</span><span class="hljs-parameter"> -VirtualNetworkName </span>vnetPS<span class="hljs-parameter"> -ResourceGroupName </span>vnetPS<span class="hljs-parameter"> -Name </span>PS2AZ
<span class="hljs-variable">$</span>ps2az.AllowForwardedTraffic = <span class="hljs-literal">$true
</span><span class="hljs-pscommand">Set-AzureRmVirtualNetworkPeering</span><span class="hljs-parameter"> -VirtualNetworkPeering </span><span class="hljs-variable">$</span>ps2az</code></pre>
<h2>Azure CLI 2.0</h2>
Avec Azure CLI, il faudra recupéré l'id du réseau distant pour le peering avec la commande :
<pre><code class="lang-bash">az network vnet list -g RG-Vnet-Exo-JSON</code></pre>
et vous recuperez un objets json, cherchez votre id qui sera du type :
[code]/subscriptions/VOTREIDDESOUSCRPTION/resourceGroups/RG-Vnet-Exo-JSON/providers/Microsoft.Network/virtualNetworks/vnet-test-json[/code]

Si vous utilisez le bash (Ubuntu on Windows par exemple) créer une variable comme ceci :
<pre><code class="lang-bash">JSON_VNET_ID="/subscriptions/VOTREIDDESOUSCRPTION/resourceGroups/RG-Vnet-Exo-JSON/providers/Microsoft.Network/virtualNetworks/vnet-test-json"</code></pre>
Ce sera beaucoup plus simple pour l'appeler dans la commande suivante :
<pre><code class="lang-bash"> az network vnet peering create --name AZ2JSON \
   --remote-vnet-id ${$JSON_VNET_ID} \
   --resource-group RG-Vnet-Exo-AZ \
   --vnet-name vnet-test-az \ 
   --allow-forwarded-traffic \
   --allow-vnet-access \
</code></pre>
Théoriquement, nous pourrions executer la commande pour effectuer le peering dans l'autre sens, mais nous allons la faire avec notre template de la partie 1.
<h2>ARM Template</h2>
Pour déclarer un vnet peering dans un template ARM, à l'heure actuelle, via le portail ou la ligne de commande (Powershell ou Azure CLI 2.0) il n'est pas possible dans le "contexte" de manipuler deux Resource Group. Du coup, il faut le faire d'un coté puis de l'autre, ou utiliser des templates "nested" c'est à dire un template "parent" puis deux "enfants". Nous n'allons (pas encore ;)) aborder ce type de template. (Stay Tuned !)

Donc, reprenons notre template et ajoutons un nouveau type de resource Azure "Microsoft.Network/virtualNetworks/virtualNetworkPeerings"
<pre><code class="lang-json">{
"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
"contentVersion": "1.0.0.0",
"parameters": {
"vnetname": {
"type": "string",
"metadata": {
"description": "Nom du Vnet"
},
"defaultValue": "vnet-test-json"
},
"vNetRemote": {
"type": "string",
"metadata": {
"description": "Nom du Vnet distant"
},
"defaultValue": "vnet-test-az"
},
"RGvNetRemote": {
"type": "string",
"metadata": {
"description": "Nom du Resource Group"
},
"defaultValue": "RG-Vnet-Exo-AZ"
},
"addressSpacePrefix": {
"type": "string",
"metadata": {
"description": "IP v4 (RFC1918)"
},
"defaultValue": "10.0.2.0/24"
},
"subnetName": {
"type": "string",
"metadata": {
"description": "Nom du Subnet"
},
"defaultValue": "jsonfrontend"
},
"subnetPrefix": {
"type": "string",
"metadata": {
"description": "IP v4 du Subnet"
},
"defaultValue": "10.0.2.0/26"
}
},
"variables": {},
"resources": [
{
"apiVersion": "2015-06-15",
"type": "Microsoft.Network/virtualNetworks",
"name": "[parameters('VnetName')]",
"location": "[resourceGroup().location]",
"tags": {
"displayName": "JSON-VNET"
},
"properties": {
"addressSpace": {
"addressPrefixes": [
"[parameters('addressSpacePrefix')]"
]
},
"subnets": [
{
"name": "[parameters('subnetName')]",
"properties": {
"addressPrefix": "[parameters('subnetPrefix')]"
}
}
]
}
},
{
"apiVersion": "2017-03-01",
"type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
"name": "[concact( parameters('VnetName') , '/JSON2AZ')]",
"location": "[resourceGroup().location]",
"properties": {
"allowVirtualNetworkAccess": true,
"allowForwardedTraffic": true,
"allowGatewayTransit": false,
"useRemoteGateways": false,
"remoteVirtualNetwork": {
"id": "[resourceId( subscription().subscriptionid, parameters('RGvNetRemote') , 'Microsoft.Network/virtualNetworks', parameters('vNetRemote'))]"
}
}
}
],
"outputs": {}
}</code></pre>
La méthode de déploiement reste la même que dans la partie 1.