---
ID: 42
post_title: 'Side loading application without SCCM &#8211; Part 3'
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2016/01/11/side-loading-application-without-sccm-part-3/
published: true
post_date: 2016-01-11 19:57:23
---
So I assume you have read the part 1 and the part 2 of this little series.
<h1>The first Function</h1>
Here is my Function called "Set-Manifest" because I create it for changing the XML and the Block Map :

[code lang="powershell"]

Function Set-Manifest {
param(
[String]
[ValidateSet("Bundle","Block","Manifest","BlockAppx")]
$Mode,
[XML]
$xmlfile,
[String]
$CertificateSAN,
[String]
$filename,
[String]
$BlockMap
)

switch ($Mode)
{
'Bundle'
{
$xmlfile.Bundle.Identity.Publisher = $CertificateSAN
$xmlfile.Save($filename)
}
'Block'
{
$xmlfile.BlockMap.File.RemoveAttribute("Size")
$xmlfile.BlockMap.File.RemoveAttribute("LfhSize")
$xmlfile.BlockMap.File.RemoveChild($xmlfile.BlockMap.File.Block)
$xmlfile.Save($filename)
}
'Manifest'
{
$xmlfile.Package.Identity.Publisher = $CertificateSAN
$xmlfile.Save($filename)
}
'BlockAppx'
{
$xmltemp = $xmlfile.BlockMap.File |? { $_.Name -eq "AppxManifest.xml"}
$xmlfile.BlockMap.RemoveChild($xmltemp)
$xmltemp.RemoveChild($xmltemp.Block)
$xmltemp.RemoveAttribute("Size")
$xmltemp.RemoveAttribute("LfhSize")
$xmlfile.BlockMap.AppendChild($xmltemp)
$xmlfile.Save($filename)
}
}
}

[/code]

I used this function like that in my "main" script:

[code lang="powershell"]

#region manifest
Write-Host "Getting the manifest in :`t $("$($destinationroot)\expanded\") !" -ForegroundColor Green
$ManifestBundle = Get-ChildItem -Filter AppxBundleManifest.xml -Path "$($destinationroot)\expanded\bundle" -Recurse
$ManifestAppxs = Get-ChildItem -Filter AppxManifest.xml -Path "$($destinationroot)\expanded\Appx\" -Recurse
#region bundle manifest
Write-Host "Settings the Bundle manifest in :`t $("$($destinationroot)\expanded\") !" -ForegroundColor Green
if($($ManifestBundle.count)){
foreach($Manifest in $ManifestBundle){
[xml]$manifestxml = Get-Content $Manifest.FullName
Set-Manifest -Mode Bundle -certificate $certificateSAN -filename $($Manifest.fullname)  -xmlfile $manifestxml | Out-Null
}
}else{
[xml]$ManifestXML = Get-Content $ManifestBundle.FullName
Set-Manifest -Mode Bundle -certificate $certificateSAN -filename $($ManifestBundle.fullname)  -xmlfile $ManifestXML | Out-Null
}
#endregion
#region Appxs Manifests
Write-Host "Settings the Appx manifest in :`t $("$($destinationroot)\expanded\") !" -ForegroundColor Green
if($($ManifestAppxs.count)){
foreach($ManifestAppx in $ManifestAppxs){
[xml]$ManifestXML = Get-Content $ManifestAppx.FullName
Set-Manifest -Mode Manifest -certificate $certificateSAN -filename $($ManifestAppx.fullname)  -xmlfile $ManifestXML | Out-Null
}
}else{
[xml]$manifestxml = Get-Content $ManifestAppx.FullName
Set-Manifest -Mode Manifest -certificate $certificateSAN -filename $($ManifestAppx.fullname)  -xmlfile $ManifestXML | Out-Null
}
#endregion
#endregion
#region bundle BlockMaps
$ManifestBundleBlockMaps = Get-ChildItem -Filter AppxBlockMap.xml -Path "$($destinationroot)\expanded\bundle" -Recurse
$ManifestAppxBlockMaps = Get-ChildItem -Filter AppxBlockMap.xml -Path "$($destinationroot)\expanded\Appx\" -Recurse
Write-Host "Settings the Bundle Block Maps in :`t $("$($destinationroot)\expanded\") !" -ForegroundColor Green
if($($ManifestBundleBlockMaps.count)){
foreach($BlockMaps in $ManifestBundleBlockMaps){
[xml]$BlockMapsXml = Get-Content $BlockMaps.FullName
Set-Manifest -Mode Block -certificate $certificateSAN -filename $($BlockMaps.fullname)  -xmlfile $BlockMapsXml | Out-Null
}
}else{
[xml]$BlockMapsXML = Get-Content $BlockMaps.FullName
Set-Manifest -Mode Block -certificate $certificateSAN -filename $($BlockMaps.fullname)  -xmlfile $BlockMapsXML | Out-Null
}
#endregion
#region Appxs BlockMaps
Write-Host "Settings the Appx Block Maps in :`t $("$($destinationroot)\expanded\") !" -ForegroundColor Green
if($($ManifestAppxBlockMaps.count)){
foreach($BlockMaps in $ManifestAppxBlockMaps){
[xml]$BlockMapsXml = Get-Content $BlockMaps.FullName
Set-Manifest -Mode BlockAppx -certificate $certificateSAN -filename $($BlockMaps.fullname)  -xmlfile $BlockMapsXML | Out-Null
}
}else{
[xml]$BlockMapsXml = Get-Content $ManifestAppxBlockMaps.FullName
Set-Manifest -Mode BlockAppx -certificate $certificateSAN -filename $($ManifestAppxBlockMaps.fullname)  -xmlfile $BlockMapsXml | Out-Null
}
#endregion
#endregion

[/code]

Part 4 is coming soon :)

Other Parts  :
<ul>
	<li><a href="http://etienne.deneuve.xyz/2016/01/11/side-loading-application-without-sccm-part-1/"><u><span style="color: #0066cc;">Part 1</span></u></a></li>
	<li><a href="http://etienne.deneuve.xyz/2016/01/11/side-loading-application-without-sccm-part-2/" target="_blank"><u><span style="color: #0066cc;">Part 2</span></u></a></li>
	<li><a href="http://etienne.deneuve.xyz/2016/01/11/side-loading-application-without-sccm-part-3/" target="_blank"><u><span style="color: #0066cc;">Part 3 (you are here)</span></u></a></li>
	<li>The full script is on my (New) Git : <a href="https://github.com/EtienneDeneuve/Powershell">Go to Git !</a></li>
</ul>