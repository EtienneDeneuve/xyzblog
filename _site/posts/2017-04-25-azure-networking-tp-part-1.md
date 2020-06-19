---
ID: 144
title: 'Azure Networking &#8211; TP &#8211; Part 1 &#8211; vNets &#038; Subnets'
author: etienne.deneuve
post_excerpt: ""
layout: layouts/post-sidebar.njk
mySlug: azure-networking-tp-part-1
permalink: "{{ page.date | date: '%Y/%m/%d' }}/{{ mySlug }}/index.html"

published: true
date: 2017-04-25 11:00:26
---
<a href="https://www.linkedin.com/in/lopesmickael">Mickael Lopes</a> et moi avons animé une session au Global Azure Bootcamp intitulée : "Le réseau dans Azure : Cas d'usage et Retour d'expériences".

Les slides de  notre session sont dispo sur slideshare ici : <a href="https://www.slideshare.net/MickaelLOPES91/gab-le-rseau-dans-azure">Slideshare de Mickael</a>

Comme prévu, voici des TP pour aller avec notre présentation, a faire vous même :)
<h1>Les vNets et Subnets</h1>
Avant d'attaquer des éléments plus complexe, nous allons apprendre a créer un VNet avec un subnet avec les trois moyens disponibles sur Azure.
<h2>Powershell</h2>
Avant de se lancer, commencez par installer votre station de travail convenablement avec la doc Microsoft qui va bien : <a href="https://docs.microsoft.com/fr-fr/powershell/azureps-cmdlets-docs">Powershell for Azure</a>

Dans Azure, on commence toujours par créer un Resource Group donc :
<pre>New-AzureRmResourceGroup `
-Name RG-Vnet-Exo-PS `
-Location northeurope</pre>
Ensuite nous allons créer un vNet :
<pre><code class="lang-powershell">$vnetexo = New-AzureRmVirtualNetwork -ResourceGroupName RG-Vnet-Exo-PS `
-Name vnet-test-ps `
-AddressPrefix 10.0.0.0/24 `
-Location northeurope</code></pre>
puis afin d'ajouter un subnet :
<pre><code class="lang-powershell">Add-AzureRmVirtualNetworkSubnetConfig -Name psfrontend `
-VirtualNetwork $vnetexo `
-AddressPrefix 10.0.0.0/26
Set-AzureRmVirtualNetwork -VirtualNetwork $vnetexo</code></pre>
<h2>Azure CLI 2.0</h2>
Comme pour la partie Powershell, suivez la doc d'installation de Azure CLI 2.0 ici : <a href="https://docs.microsoft.com/en-us/cli/azure/install-azure-cli">Azure CLI 2.0</a>
Comme dans l'exemple précédent nous allons créer un Resource Group :
<pre><code class="lang-bash">az group create \
--name RG-Vnet-Exo-AZ \
--location northeurope
</code></pre>
Ensuite nous allons créer un vNet et le subnet :
<pre><code class="lang-bash">az network vnet create \
--name vnet-test-az \
--resource-group RG-Vnet-Exo-AZ \
--location northeurope \
--address-prefixes 10.0.1.0/24 \
--subnet-name azfrontend \
--subnet-prefixes 10.0.1.0/26
</code></pre>
<h2>ARM Template</h2>
Pour commencer, utilisez un éditeur de texte un peu évoluer, le JSON sans outil, ca pique un peu... Je vous recommande Visual Studio Code, avec ce petit article : <a href="https://etienne.deneuve.xyz/2017/01/26/visual-studio-code-pour-ansible-terraform/" target="_blank" rel="noopener noreferrer">sur mon blog</a>
Un Template Json simple contient plusieurs parties : "variables", "parameters", "resources" et "outputs". Sans réexpliquer l'ensemble, voici à quoi notre Template va ressembler :
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
    "vnetname": {
      "type": "string",
      "metadata": {
        "description": "Nom du Vnet"
      },
      "defaultValue": "vnet-test-json"
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
    }
  ],
  "outputs": {}
}</code></pre>
Ensuite pour déployer ce fichier soit via le portail avec la doc ici : <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-deploy-portal">Azure Template &amp; Portal</a>
ou alors via Powershell et Azure CLI 2.0 :
<h5>Powershell :</h5>
<pre>New-AzureRmResourceGroup -Name RG-Vnet-Exo-JSON `
   -Location " North Europe"
New-AzureRmResourceGroupDeployment -Name RG-Vnet-Exo-JSON `
   -ResourceGroupName ExampleResourceGroup `
   -TemplateFile c:\MyTemplates\vnet-simple.json</pre>
<h5>Azure CLI 2.0:</h5>
<pre>az group create --name RG-XXX --location "North Europe“
az group deployment create \
--name ExampleDeployment \
--resource-group ExampleGroup \
--template-file vnet-simple.json</pre>