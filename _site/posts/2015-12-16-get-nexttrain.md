---
ID: 12
title: Get-NextTrain !
author: etienne.deneuve
post_excerpt: ""
layout: layouts/post-sidebar.njk
mySlug: get-nexttrain
permalink: "{{ page.date | date: '%Y/%m/%d' }}/{{ mySlug }}/index.html"

published: true
date: 2015-12-16 13:44:50
---

# Let's play with SNCF Api and Powershell !

My goal was to get a list of the next train in the train station near my home, from powershell...

My functions are below, feel free to leave a comment, copy it, it's free ! (Now on Github ! [git it !](https://github.com/EtienneDeneuve/Powershell/blob/master/GetNextTrain/TheScript.ps1)</a>)

Of course, it's not a serious post, but more is coming ;)

You can try this :

``` powershell
(Get-gare -gare "clamart").idgare | Get-TrainDirection |%{ Get-NextTrain -idgare $_.idgare -traindirection $_.direction } | FT
ETADeparture Direction ETAArrival ETAinMin From GareName
------------ --------- ---------- -------- ---- --------
16/12/2015 13:49:00 Paris-Montparnasse 1-2 (Paris) 16/12/2015 13:49:00 0:1:18 Rambouillet vers Paris-Montparnasse 1-2 gare de Clamart
16/12/2015 14:04:00 Paris-Montparnasse 1-2 (Paris) 16/12/2015 14:04:00 0:16:18 Mantes-la-Jolie vers Paris-Montparnasse 1-2 gare de Clamart
16/12/2015 14:19:00 Paris-Montparnasse 1-2 (Paris) 16/12/2015 14:19:00 0:31:18 Rambouillet vers Paris-Montparnasse 1-2 gare de Clamart
16/12/2015 14:34:00 Paris-Montparnasse 1-2 (Paris) 16/12/2015 14:34:00 0:46:18 Plaisir-Grignon vers Paris-Montparnasse 1-2 gare de Clamart
16/12/2015 14:49:00 Paris-Montparnasse 1-2 (Paris) 16/12/2015 14:49:00 1:1:18 Rambouillet vers Paris-Montparnasse 1-2 gare de Clamart
16/12/2015 13:57:00 Mantes-la-Jolie (Mantes-la-Jolie) 16/12/2015 13:57:00 0:9:18 Paris-Montparnasse 1-2 vers Mantes-la-Jolie gare de Clamart
16/12/2015 14:57:00 Mantes-la-Jolie (Mantes-la-Jolie) 16/12/2015 14:57:00 1:9:18 Paris-Montparnasse 1-2 vers Mantes-la-Jolie gare de Clamart
16/12/2015 14:12:00 Rambouillet (Rambouillet) 16/12/2015 14:12:00 0:24:17 Paris-Montparnasse 1-2 vers Rambouillet gare de Clamart
16/12/2015 14:42:00 Rambouillet (Rambouillet) 16/12/2015 14:42:00 0:54:17 Paris-Montparnasse 1-2 vers Rambouillet gare de Clamart
16/12/2015 14:27:00 Plaisir-Grignon (Plaisir) 16/12/2015 14:27:00 0:39:18 Paris-Montparnasse 1-2 vers Plaisir-Grignon gare de Clamart
```

``` powershell
Function Get-NextTrain
{
[cmdletbinding()]
param(
[Parameter(Mandatory = $true, ValueFromPipeline=$true)]
$idgare,
[Parameter(Mandatory = $true, ValueFromPipeline=$true)]
$traindirection,
$apikey = 'YourApiKeyfromapisncfwebsite'
)

$datenow = Get-date -Format yyyyMMddTHHmmss
$params2 = @{uri = "https://api.sncf.com/v1/coverage/sncf/stop_areas/$($idgare)/departures?from_datetime=$($datenow)";

               Method = 'Get'; #(or POST, or whatever)
               Headers = @{Authorization = 'Basic ' + [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes("$($apikey):"));
       } #end headers hash table
   } #end $params hash table
$string = @"
Liste des prochains trains pour {0} en provenance de {1} doit arriver a la {2} a {3} et partir a {4}
"@
$var2 = invoke-restmethod @params2
$alltrain = ,@()
foreach($departure in $($var2.departures)){
   if($departure.display_informations.direction -like $($traindirection)){
    $direction = $($departure.display_informations.direction -replace "gare de ","") -replace " 1-2 (Paris)",""
    [datetime]$convertarrival = ConvertTo-SncfDateTime -sncfdate $($departure.stop_date_time.arrival_date_time)
    [datetime]$convertdeparture = ConvertTo-SncfDateTime -sncfdate $($departure.stop_date_time.departure_date_time)
    [timespan]$timefromnow = NEW-TIMESPAN –Start $(Get-date) –End $convertarrival
    $hash = @{
        Direction = $($direction)
        From = $($departure.route.name)
        GareName = $($departure.stop_point.name)
        ETAArrival = $($convertarrival)
        ETADeparture =  $($convertdeparture)
        ETAinMin =  "$($timefromnow.Hours):$($timefromnow.Minutes):$($timefromnow.Seconds)"
    }
        $Object = New-Object PSObject -Property $hash
        $alltrain += $Object
     }
}
return $alltrain
}

Function Get-TrainDirection
{
 [cmdletbinding()]
param(
[Parameter(Mandatory = $true, ValueFromPipeline=$true)]
[string]$idgare,
$apikey = 'YourApiKeyfromapisncfwebsite'
)
$datenow = Get-date -Format yyyyMMddTHHmmss
$params2 = @{uri = "https://api.sncf.com/v1/coverage/sncf/stop_areas/$($idgare)/departures?from_datetime=$($datenow)";

                   Method = 'Get'; #(or POST, or whatever)
                   Headers = @{Authorization = 'Basic ' + [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes("$($apikey):"));
           } #end headers hash table
   } #end $params hash table
$var2 = invoke-restmethod @params2
$direction = ,@()
foreach($departure in $($var2.departures)){
    $hash = @{
        Direction = $($departure.display_informations.direction.ToString())
        idgare = $($idgare)
    }
    $Object = New-Object PSObject -Property $hash
    $direction += $Object
}
$direction | Select-Object idgare,direction -Unique  -Skip 1
}

Function Get-Gare
{
param(
$gare,
$apikey = 'YourApiKeyfromapisncfwebsite'
)
$params = @{uri = "https://api.sncf.com/v1/coverage/sncf/places?q=$($gare)";
               Method = 'Get'; #(or POST, or whatever)
               Headers = @{Authorization = 'Basic ' + [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes("$($apikey):"));
       } #end headers hash table
   } #end $params hash table
$var = invoke-restmethod @params
foreach($stoparea in $($var.places |?{ $_.embedded_type -eq "stop_area"})){
 $hash = @{
        Direction = $($stoparea.Name)
        idgare = $($stoparea.id)
    }
    $Object = New-Object PSObject -Property $hash
    Write-Output $Object
}

}

function ConvertTo-SncfDateTime
{
param(
    $sncfdate
)
$sncfyear = $sncfdate.Substring(0,4)
$sncfmonth = $sncfdate.Substring(4,2)
$sncfday = $sncfdate.Substring(6,2)
$sncfhour = $sncfdate.Substring(9,2)
$sncfminute = $sncfdate.Substring(11,2)
$sncfsecond = $sncfdate.Substring(13,2)

$datetime = Get-date -Year $sncfyear -Day $sncfday -Month $sncfmonth -Hour $sncfhour -Second $sncfsecond -Minute $sncfminute
Write-Output $datetime
}
```
