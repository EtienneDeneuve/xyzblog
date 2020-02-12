---
ID: 12
post_title: Get-NextTrain !
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2015/12/16/get-nexttrain/
published: true
post_date: 2015-12-16 13:44:50
---
<h1><a href="https://etienne.deneuve.xyz/wp-content/uploads/2015/12/SNCF_Logo2011.jpg" rel="attachment wp-att-14"><img class="alignnone size-medium wp-image-14" src="http://40.127.173.162/wp-content/uploads/2015/12/SNCF_Logo2011-300x158.jpg" alt="SNCF_Logo2011" width="300" height="158" /></a></h1>
<h1>Let's play with SNCF Api and Powershell !</h1>
My goal was to get a list of the next train in the train station near my home, from powershell...

My functions are below, feel free to leave a comment, copy it, it's free ! (Now on Github ! <a href="https://github.com/EtienneDeneuve/Powershell/blob/master/GetNextTrain/TheScript.ps1" target="_blank" rel="noopener">git it !</a>)

Of course, it's not a serious post, but more is coming ;)

You can try this :

&nbsp;
<pre>PS C:\&gt; (Get-gare -gare "clamart").idgare | Get-TrainDirection |%{ Get-NextTrain -idgare $_.idgare -traindirection $_.direction } | FT
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
</pre>
&nbsp;
<pre class="PowerShellColorizedScript"><span style="color: #00008b;">Function</span> <span style="color: #8a2be2;">Get-NextTrain</span>
<span style="color: #000000;">{</span>
<span style="color: #a9a9a9;">[</span><span style="color: #00bfff;">cmdletbinding</span><span style="color: #000000;">(</span><span style="color: #000000;">)</span><span style="color: #a9a9a9;">]</span>
    <span style="color: #00008b;">param</span><span style="color: #000000;">(</span>
    <span style="color: #a9a9a9;">[</span><span style="color: #00bfff;">Parameter</span><span style="color: #000000;">(</span><span style="color: #000000;">Mandatory</span> <span style="color: #a9a9a9;">=</span> <span style="color: #ff4500;">$true</span><span style="color: #a9a9a9;">,</span> <span style="color: #000000;">ValueFromPipeline</span><span style="color: #a9a9a9;">=</span><span style="color: #ff4500;">$true</span><span style="color: #000000;">)</span><span style="color: #a9a9a9;">]</span>
    <span style="color: #ff4500;">$idgare</span><span style="color: #a9a9a9;">,</span>
    <span style="color: #a9a9a9;">[</span><span style="color: #00bfff;">Parameter</span><span style="color: #000000;">(</span><span style="color: #000000;">Mandatory</span> <span style="color: #a9a9a9;">=</span> <span style="color: #ff4500;">$true</span><span style="color: #a9a9a9;">,</span> <span style="color: #000000;">ValueFromPipeline</span><span style="color: #a9a9a9;">=</span><span style="color: #ff4500;">$true</span><span style="color: #000000;">)</span><span style="color: #a9a9a9;">]</span>
    <span style="color: #ff4500;">$traindirection</span><span style="color: #a9a9a9;">,</span>
    <span style="color: #ff4500;">$apikey</span> <span style="color: #a9a9a9;">=</span> <span style="color: #8b0000;">'YourApiKeyfromapisncfwebsite'</span>
<span style="color: #000000;">)</span>

<span style="color: #ff4500;">$datenow</span> <span style="color: #a9a9a9;">=</span> <span style="color: #0000ff;">Get-date</span> <span style="color: #000080;">-Format</span> yyyyMMddTHHmmss
<span style="color: #ff4500;">$params2</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">@{</span><span style="color: #000000;">uri</span> <span style="color: #a9a9a9;">=</span> <span style="color: #8b0000;">"https://api.sncf.com/v1/coverage/sncf/stop_areas/$($idgare)/departures?from_datetime=$($datenow)"</span><span style="color: #000000;">;</span>

                   <span style="color: #000000;">Method</span> <span style="color: #a9a9a9;">=</span> <span style="color: #8b0000;">'Get'</span><span style="color: #000000;">;</span> <span style="color: #006400;">#(or POST, or whatever)</span>
                   <span style="color: #000000;">Headers</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">@{</span><span style="color: #000000;">Authorization</span> <span style="color: #a9a9a9;">=</span> <span style="color: #8b0000;">'Basic '</span> <span style="color: #a9a9a9;">+</span> <span style="color: #008080;">[Convert]</span><span style="color: #a9a9a9;">::</span><span style="color: #000000;">ToBase64String</span><span style="color: #000000;">(</span><span style="color: #008080;">[Text.Encoding]</span><span style="color: #a9a9a9;">::</span><span style="color: #000000;">ASCII</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">GetBytes</span><span style="color: #000000;">(</span><span style="color: #8b0000;">"$($apikey):"</span><span style="color: #000000;">)</span><span style="color: #000000;">)</span><span style="color: #000000;">;</span>
           <span style="color: #000000;">}</span> <span style="color: #006400;">#end headers hash table</span>
   <span style="color: #000000;">}</span> <span style="color: #006400;">#end $params hash table</span>
<span style="color: #ff4500;">$string</span> <span style="color: #a9a9a9;">=</span> <span style="color: #8b0000;">@"
    Liste des prochains trains pour {0} en provenance de {1} doit arriver a la {2} a {3} et partir a {4}
"@</span>
<span style="color: #ff4500;">$var2</span> <span style="color: #a9a9a9;">=</span> <span style="color: #0000ff;">invoke-restmethod</span> <span style="color: #ff4500;">@params2</span>
<span style="color: #ff4500;">$alltrain</span> <span style="color: #a9a9a9;">=</span> <span style="color: #a9a9a9;">,</span><span style="color: #000000;">@(</span><span style="color: #000000;">)</span>
<span style="color: #00008b;">foreach</span><span style="color: #000000;">(</span><span style="color: #ff4500;">$departure</span> <span style="color: #00008b;">in</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$var2</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">departures</span><span style="color: #000000;">)</span><span style="color: #000000;">)</span><span style="color: #000000;">{</span>
   <span style="color: #00008b;">if</span><span style="color: #000000;">(</span><span style="color: #ff4500;">$departure</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">display_informations</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">direction</span> <span style="color: #a9a9a9;">-like</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$traindirection</span><span style="color: #000000;">)</span><span style="color: #000000;">)</span><span style="color: #000000;">{</span>
        <span style="color: #ff4500;">$direction</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$departure</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">display_informations</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">direction</span> <span style="color: #a9a9a9;">-replace</span> <span style="color: #8b0000;">"gare de "</span><span style="color: #a9a9a9;">,</span><span style="color: #8b0000;">""</span><span style="color: #000000;">)</span> <span style="color: #a9a9a9;">-replace</span> <span style="color: #8b0000;">" 1-2 (Paris)"</span><span style="color: #a9a9a9;">,</span><span style="color: #8b0000;">""</span>
        <span style="color: #008080;">[datetime]</span><span style="color: #ff4500;">$convertarrival</span> <span style="color: #a9a9a9;">=</span> <span style="color: #0000ff;">ConvertTo-SncfDateTime</span> <span style="color: #000080;">-sncfdate</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$departure</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">stop_date_time</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">arrival_date_time</span><span style="color: #000000;">)</span>
        <span style="color: #008080;">[datetime]</span><span style="color: #ff4500;">$convertdeparture</span> <span style="color: #a9a9a9;">=</span> <span style="color: #0000ff;">ConvertTo-SncfDateTime</span> <span style="color: #000080;">-sncfdate</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$departure</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">stop_date_time</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">departure_date_time</span><span style="color: #000000;">)</span>
        <span style="color: #008080;">[timespan]</span><span style="color: #ff4500;">$timefromnow</span> <span style="color: #a9a9a9;">=</span> <span style="color: #0000ff;">NEW-TIMESPAN</span> <span style="color: #000080;">–Start</span> <span style="color: #000000;">$(</span><span style="color: #0000ff;">Get-date</span><span style="color: #000000;">)</span> <span style="color: #000080;">–End</span> <span style="color: #ff4500;">$convertarrival</span>
        <span style="color: #ff4500;">$hash</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">@{</span>
            <span style="color: #000000;">Direction</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$direction</span><span style="color: #000000;">)</span>
            <span style="color: #000000;">From</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$departure</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">route</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">name</span><span style="color: #000000;">)</span>
            <span style="color: #000000;">GareName</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$departure</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">stop_point</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">name</span><span style="color: #000000;">)</span>
            <span style="color: #000000;">ETAArrival</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$convertarrival</span><span style="color: #000000;">)</span>
            <span style="color: #000000;">ETADeparture</span> <span style="color: #a9a9a9;">=</span>  <span style="color: #000000;">$(</span><span style="color: #ff4500;">$convertdeparture</span><span style="color: #000000;">)</span>
            <span style="color: #000000;">ETAinMin</span> <span style="color: #a9a9a9;">=</span>  <span style="color: #8b0000;">"$($timefromnow.Hours):$($timefromnow.Minutes):$($timefromnow.Seconds)"</span>
        <span style="color: #000000;">}</span>
            <span style="color: #ff4500;">$Object</span> <span style="color: #a9a9a9;">=</span> <span style="color: #0000ff;">New-Object</span> <span style="color: #8a2be2;">PSObject</span> <span style="color: #000080;">-Property</span> <span style="color: #ff4500;">$hash</span>
            <span style="color: #ff4500;">$alltrain</span> <span style="color: #a9a9a9;">+=</span> <span style="color: #ff4500;">$Object</span>
         <span style="color: #000000;">}</span>
<span style="color: #000000;">}</span>
<span style="color: #00008b;">return</span> <span style="color: #ff4500;">$alltrain</span>
<span style="color: #000000;">}</span>

<span style="color: #00008b;">Function</span> <span style="color: #8a2be2;">Get-TrainDirection</span>
<span style="color: #000000;">{</span>
     <span style="color: #a9a9a9;">[</span><span style="color: #00bfff;">cmdletbinding</span><span style="color: #000000;">(</span><span style="color: #000000;">)</span><span style="color: #a9a9a9;">]</span>
    <span style="color: #00008b;">param</span><span style="color: #000000;">(</span>
    <span style="color: #a9a9a9;">[</span><span style="color: #00bfff;">Parameter</span><span style="color: #000000;">(</span><span style="color: #000000;">Mandatory</span> <span style="color: #a9a9a9;">=</span> <span style="color: #ff4500;">$true</span><span style="color: #a9a9a9;">,</span> <span style="color: #000000;">ValueFromPipeline</span><span style="color: #a9a9a9;">=</span><span style="color: #ff4500;">$true</span><span style="color: #000000;">)</span><span style="color: #a9a9a9;">]</span>
    <span style="color: #008080;">[string]</span><span style="color: #ff4500;">$idgare</span><span style="color: #a9a9a9;">,</span>
    <span style="color: #ff4500;">$apikey</span> <span style="color: #a9a9a9;">=</span> <span style="color: #8b0000;">'<span style="color: #8b0000;">YourApiKeyfromapisncfwebsite</span>'</span>
    <span style="color: #000000;">)</span>
    <span style="color: #ff4500;">$datenow <span style="color: #a9a9a9;">=</span> <span style="color: #0000ff;">Get-date</span> <span style="color: #000080;">-Format</span> yyyyMMddTHHmmss
$params2</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">@{</span><span style="color: #000000;">uri</span> <span style="color: #a9a9a9;">=</span> <span style="color: #8b0000;">"https://api.sncf.com/v1/coverage/sncf/stop_areas/$($idgare)/departures?from_datetime=$($datenow)"</span><span style="color: #000000;">;</span>

                       <span style="color: #000000;">Method</span> <span style="color: #a9a9a9;">=</span> <span style="color: #8b0000;">'Get'</span><span style="color: #000000;">;</span> <span style="color: #006400;">#(or POST, or whatever)</span>
                       <span style="color: #000000;">Headers</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">@{</span><span style="color: #000000;">Authorization</span> <span style="color: #a9a9a9;">=</span> <span style="color: #8b0000;">'Basic '</span> <span style="color: #a9a9a9;">+</span> <span style="color: #008080;">[Convert]</span><span style="color: #a9a9a9;">::</span><span style="color: #000000;">ToBase64String</span><span style="color: #000000;">(</span><span style="color: #008080;">[Text.Encoding]</span><span style="color: #a9a9a9;">::</span><span style="color: #000000;">ASCII</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">GetBytes</span><span style="color: #000000;">(</span><span style="color: #8b0000;">"$($apikey):"</span><span style="color: #000000;">)</span><span style="color: #000000;">)</span><span style="color: #000000;">;</span>
               <span style="color: #000000;">}</span> <span style="color: #006400;">#end headers hash table</span>
       <span style="color: #000000;">}</span> <span style="color: #006400;">#end $params hash table</span>
    <span style="color: #ff4500;">$var2</span> <span style="color: #a9a9a9;">=</span> <span style="color: #0000ff;">invoke-restmethod</span> <span style="color: #ff4500;">@params2</span>
    <span style="color: #ff4500;">$direction</span> <span style="color: #a9a9a9;">=</span> <span style="color: #a9a9a9;">,</span><span style="color: #000000;">@(</span><span style="color: #000000;">)</span>
    <span style="color: #00008b;">foreach</span><span style="color: #000000;">(</span><span style="color: #ff4500;">$departure</span> <span style="color: #00008b;">in</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$var2</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">departures</span><span style="color: #000000;">)</span><span style="color: #000000;">)</span><span style="color: #000000;">{</span>
        <span style="color: #ff4500;">$hash</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">@{</span>
            <span style="color: #000000;">Direction</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$departure</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">display_informations</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">direction</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">ToString</span><span style="color: #000000;">(</span><span style="color: #000000;">)</span><span style="color: #000000;">)</span>
            <span style="color: #000000;">idgare</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$idgare</span><span style="color: #000000;">)</span>
        <span style="color: #000000;">}</span>
        <span style="color: #ff4500;">$Object</span> <span style="color: #a9a9a9;">=</span> <span style="color: #0000ff;">New-Object</span> <span style="color: #8a2be2;">PSObject</span> <span style="color: #000080;">-Property</span> <span style="color: #ff4500;">$hash</span>
        <span style="color: #ff4500;">$direction</span> <span style="color: #a9a9a9;">+=</span> <span style="color: #ff4500;">$Object</span>
    <span style="color: #000000;">}</span>
    <span style="color: #ff4500;">$direction</span> <span style="color: #a9a9a9;">|</span> <span style="color: #0000ff;">Select-Object</span> <span style="color: #8a2be2;">idgare</span><span style="color: #a9a9a9;">,</span><span style="color: #8a2be2;">direction</span> <span style="color: #000080;">-Unique</span>  <span style="color: #000080;">-Skip</span> <span style="color: #800080;">1</span>
<span style="color: #000000;">}</span>

<span style="color: #00008b;">Function</span> <span style="color: #8a2be2;">Get-Gare</span>
<span style="color: #000000;">{</span>
<span style="color: #00008b;">param</span><span style="color: #000000;">(</span>
    <span style="color: #ff4500;">$gare</span><span style="color: #a9a9a9;">,</span>
    <span style="color: #ff4500;">$apikey</span> <span style="color: #a9a9a9;">=</span> <span style="color: #8b0000;">'<span style="color: #8b0000;">YourApiKeyfromapisncfwebsite</span>'</span>
<span style="color: #000000;">)</span>
<span style="color: #ff4500;">$params</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">@{</span><span style="color: #000000;">uri</span> <span style="color: #a9a9a9;">=</span> <span style="color: #8b0000;">"https://api.sncf.com/v1/coverage/sncf/places?q=$($gare)"</span><span style="color: #000000;">;</span>
                   <span style="color: #000000;">Method</span> <span style="color: #a9a9a9;">=</span> <span style="color: #8b0000;">'Get'</span><span style="color: #000000;">;</span> <span style="color: #006400;">#(or POST, or whatever)</span>
                   <span style="color: #000000;">Headers</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">@{</span><span style="color: #000000;">Authorization</span> <span style="color: #a9a9a9;">=</span> <span style="color: #8b0000;">'Basic '</span> <span style="color: #a9a9a9;">+</span> <span style="color: #008080;">[Convert]</span><span style="color: #a9a9a9;">::</span><span style="color: #000000;">ToBase64String</span><span style="color: #000000;">(</span><span style="color: #008080;">[Text.Encoding]</span><span style="color: #a9a9a9;">::</span><span style="color: #000000;">ASCII</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">GetBytes</span><span style="color: #000000;">(</span><span style="color: #8b0000;">"$($apikey):"</span><span style="color: #000000;">)</span><span style="color: #000000;">)</span><span style="color: #000000;">;</span>
           <span style="color: #000000;">}</span> <span style="color: #006400;">#end headers hash table</span>
   <span style="color: #000000;">}</span> <span style="color: #006400;">#end $params hash table</span>
<span style="color: #ff4500;">$var</span> <span style="color: #a9a9a9;">=</span> <span style="color: #0000ff;">invoke-restmethod</span> <span style="color: #ff4500;">@params</span>
<span style="color: #00008b;">foreach</span><span style="color: #000000;">(</span><span style="color: #ff4500;">$stoparea</span> <span style="color: #00008b;">in</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$var</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">places</span> <span style="color: #a9a9a9;">|</span><span style="color: #0000ff;">?</span><span style="color: #000000;">{</span> <span style="color: #ff4500;">$_</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">embedded_type</span> <span style="color: #a9a9a9;">-eq</span> <span style="color: #8b0000;">"stop_area"</span><span style="color: #000000;">}</span><span style="color: #000000;">)</span><span style="color: #000000;">)</span><span style="color: #000000;">{</span>
     <span style="color: #ff4500;">$hash</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">@{</span>
            <span style="color: #000000;">Direction</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$stoparea</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">Name</span><span style="color: #000000;">)</span>
            <span style="color: #000000;">idgare</span> <span style="color: #a9a9a9;">=</span> <span style="color: #000000;">$(</span><span style="color: #ff4500;">$stoparea</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">id</span><span style="color: #000000;">)</span>
        <span style="color: #000000;">}</span>
        <span style="color: #ff4500;">$Object</span> <span style="color: #a9a9a9;">=</span> <span style="color: #0000ff;">New-Object</span> <span style="color: #8a2be2;">PSObject</span> <span style="color: #000080;">-Property</span> <span style="color: #ff4500;">$hash</span>
        <span style="color: #0000ff;">Write-Output</span> <span style="color: #ff4500;">$Object</span>
<span style="color: #000000;">}</span>

<span style="color: #000000;">}</span>

<span style="color: #00008b;">function</span> <span style="color: #8a2be2;">ConvertTo-SncfDateTime</span>
<span style="color: #000000;">{</span>
    <span style="color: #00008b;">param</span><span style="color: #000000;">(</span>
        <span style="color: #ff4500;">$sncfdate</span>
    <span style="color: #000000;">)</span>
    <span style="color: #ff4500;">$sncfyear</span> <span style="color: #a9a9a9;">=</span> <span style="color: #ff4500;">$sncfdate</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">Substring</span><span style="color: #000000;">(</span><span style="color: #800080;">0</span><span style="color: #a9a9a9;">,</span><span style="color: #800080;">4</span><span style="color: #000000;">)</span>
    <span style="color: #ff4500;">$sncfmonth</span> <span style="color: #a9a9a9;">=</span> <span style="color: #ff4500;">$sncfdate</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">Substring</span><span style="color: #000000;">(</span><span style="color: #800080;">4</span><span style="color: #a9a9a9;">,</span><span style="color: #800080;">2</span><span style="color: #000000;">)</span>
    <span style="color: #ff4500;">$sncfday</span> <span style="color: #a9a9a9;">=</span> <span style="color: #ff4500;">$sncfdate</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">Substring</span><span style="color: #000000;">(</span><span style="color: #800080;">6</span><span style="color: #a9a9a9;">,</span><span style="color: #800080;">2</span><span style="color: #000000;">)</span>
    <span style="color: #ff4500;">$sncfhour</span> <span style="color: #a9a9a9;">=</span> <span style="color: #ff4500;">$sncfdate</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">Substring</span><span style="color: #000000;">(</span><span style="color: #800080;">9</span><span style="color: #a9a9a9;">,</span><span style="color: #800080;">2</span><span style="color: #000000;">)</span>
    <span style="color: #ff4500;">$sncfminute</span> <span style="color: #a9a9a9;">=</span> <span style="color: #ff4500;">$sncfdate</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">Substring</span><span style="color: #000000;">(</span><span style="color: #800080;">11</span><span style="color: #a9a9a9;">,</span><span style="color: #800080;">2</span><span style="color: #000000;">)</span>
    <span style="color: #ff4500;">$sncfsecond</span> <span style="color: #a9a9a9;">=</span> <span style="color: #ff4500;">$sncfdate</span><span style="color: #a9a9a9;">.</span><span style="color: #000000;">Substring</span><span style="color: #000000;">(</span><span style="color: #800080;">13</span><span style="color: #a9a9a9;">,</span><span style="color: #800080;">2</span><span style="color: #000000;">)</span>

    <span style="color: #ff4500;">$datetime</span> <span style="color: #a9a9a9;">=</span> <span style="color: #0000ff;">Get-date</span> <span style="color: #000080;">-Year</span> <span style="color: #ff4500;">$sncfyear</span> <span style="color: #000080;">-Day</span> <span style="color: #ff4500;">$sncfday</span> <span style="color: #000080;">-Month</span> <span style="color: #ff4500;">$sncfmonth</span> <span style="color: #000080;">-Hour</span> <span style="color: #ff4500;">$sncfhour</span> <span style="color: #000080;">-Second</span> <span style="color: #ff4500;">$sncfsecond</span> <span style="color: #000080;">-Minute</span> <span style="color: #ff4500;">$sncfminute</span>
    <span style="color: #0000ff;">Write-Output</span> <span style="color: #ff4500;">$datetime</span>
<span style="color: #000000;">}</span>

</pre>
