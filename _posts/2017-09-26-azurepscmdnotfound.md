---
ID: 224
post_title: 'Azure Powershell &#8211; Comment on fait quand ca n&#8217;existe pas ?'
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2017/09/26/azurepscmdnotfound/
published: true
post_date: 2017-09-26 21:16:06
---
# La vie dans Azure

Bon, admettons, vous avez besoin de filter vos VM avec le type d'OS déployé dessus afin de savoir si vous avez plus de Linux ou de Windows dans Azure.

Avec Powershell, on se dit "ouais, c'est facile". Un petit coup de Get-AzureRMVM et hop...

<img class="alignnone size-full wp-image-275" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/09/get-cazurevmos-1-gvm.png" alt="" width="737" height="104" />

Cool, "OsType", pile ce qu'on cherche, aller hop, on filtre avec | Select-Object

<img class="alignnone size-full wp-image-276" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/09/get-cazurevmos-2-so.png" alt="" width="705" height="84" />

et ouais, c'est tout vide ! Et l'aide elle dit quoi ? Pas grand chose de plus :

<img class="alignnone size-full wp-image-277" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/09/get-cazurevmos-3-help.png" alt="" width="976" height="310" />

On a plus le choix, on y va!

&gt; On va dedans !

## Vérification des membres

Avec un petit Get-Member on liste les membres de la commande Get-AzureRMVM

<img class="alignnone size-full wp-image-278" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/09/get-cazurevmos-4-gm.png" alt="" width="1188" height="486" />

On voit que "OsType" n'existe pas, c'est donc une commande qui utilise une propriété calculée.
En revanche un élément , OsProfile, parait correspondre au besoin.

<img class="alignnone size-full wp-image-279" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/09/get-cazurevmos-5-osp.png" alt="" width="796" height="24" />

On tente le Select-Object ?

<img class="alignnone size-full wp-image-280" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/09/get-cazurevmos-6-os.png" alt="" width="582" height="83" />

Raté, ce coquin est un autre objet ! Etendons le !

<img class="alignnone size-full wp-image-281" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/09/get-cazurevmos-7-exp.png" alt="" width="737" height="161" />

Super! On a trouvé un truc!

## On script ?

Déjà, on sait de base que le Script, on va s'en servir au moins une fois ;) donc pour le construire, il faut se poser quelques bonnes questions.

1. Connexion a Azure ?
2. Son nom (qui claque !)

### Le nom j'ai déjà choisi (après tout...) :

```Powershell
Get-cAzureRMVMOs
```

&gt;Personnellement, j'essaye de mettre un indicateur "c" pour "custom" sur mes functions pour les différencier des autres.

### La connexion à Azure

J'utilise un petit bout de script que j'ai récupéré (je ne sais plus où, si l'auteur se manifeste !)

```PowerShell
function Check-AzureRMSession () {
$Error.Clear()
#if context already exist
try {
Get-AzureRmVM -ErrorAction Stop | Out-Null
}
catch [System.Management.Automation.PSInvalidOperationException] {
Login-AzureRmAccount
}
$Error.Clear();
}
```

En gros, c'est simple, si on est connecté, il se passe rien, sinon on nous demande notre login. Simple, Efficace...

### Le script en lui même

```Powershell
Function Get-cAzureRMVMOs {
[CmdletBinding()]
param(

)
Check-AzureRMSession
$vms = get-azurermvm
foreach ($vm in $vms) {
$osprofile = $($vm.OSProfile)
if ($($osprofile.LinuxConfiguration) -eq $null) {
$OsType = &quot;Windows&quot;
}
elseif ($($osprofile.WindowsConfiguration) -eq $null) {
$OsType = &quot;Linux&quot;
}
else {
$OsType = $osprofile
}
[PSCustomObject]@{
AvailabilitySetReference = $vm.AvailabilitySetReference
DiagnosticsProfile = $vm.DiagnosticsProfile
DisplayHint = $vm.DisplayHint
Extensions = $vm.Extensions
HardwareProfile = $vm.HardwareProfile
Id = $vm.Id
Identity = $vm.Identity
InstanceView = $vm.InstanceView
LicenseType = $vm.LicenseType
Location = $vm.Location
Name = $vm.Name
NetworkProfile = $vm.NetworkProfile
OSType = $OsType
Plan = $vm.Plan
ProvisioningState = $vm.ProvisioningState
RequestId = $vm.RequestId
ResourceGroupName = $vm.ResourceGroupName
StatusCode = $vm.StatusCode
StorageProfile = $vm.StorageProfile
Tags = $vm.Tags
Type = $vm.Type
VmId = $vm.VmId
}
}
}
```
<h3>Résultat ?</h3>
<img class="alignnone size-full wp-image-282" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/09/get-cazurevmos-8-Cool.png" alt="" width="1168" height="487" />

Ca fonctionne ! et le filtre ?

<img class="alignnone size-full wp-image-283" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/09/get-cazurevmos-9-youpi.png" alt="" width="1167" height="436" />

Du coup, avec notre super fonction, on peut filtrer comme on voulait :)

```Powershell

Get-cAzureRMVMOsType |?{ $_.OsType -eq &quot;Windows&quot; }

```

&nbsp;
<blockquote>PS: Ce sript fonctionne très bien avec AzureRM.NetCore sur macOs :<img class="alignnone size-full wp-image-285" style="font-size: 18px;" src="https://etienne.deneuve.xyz/wp-content/uploads/2017/09/get-cazurevmos-10-youpi.png" alt="" width="1058" height="196" /><span style="font-size: 18px;">  </span></blockquote>