---
ID: 24
title: 'Side loading application without SCCM &#8211; Part 1'
author: etienne.deneuve
post_excerpt: ""
layout: layouts/post-sidebar.njk
mySlug: side-loading-application-without-sccm-part-1
permalink: "{{ page.date | date: '%Y/%m/%d' }}/{{ mySlug }}/index.html"

published: true
date: 2016-01-11 17:20:38
---
I publish my way to install side loaded application to Windows 8 and above. I looked on Internet and found nothing to install theses apps  on  computers in a company which doesn't have SCCM. I've written a powershell script which will do the work but I want to share the way I built it, instead of sharing it without any kind of explanation. This blog post is the first one of the series for passing to a long and non interesting tasks to a near complete automated one. You will need makeappx and signtool on the computer to run the script or make the process manually. It's preferable to make the process one time before launching the script. It's easier to debug something you know (yes ! ;)).

<h1>Applications for Windows Modern UI.</h1>
Let's start this post by a little explanation about Windows Sideloading Mechanics. When you build application with tools like Adobe DPS, you will get a "appxbundle". For side loading you need sign the appxbundle and the appx in the bundle. So, you need to "unbundle" the bundle, and then unpack the appx. Unfortunately you need to modify the AppManifest.xml and the BlockMap.xml (a file where all the hash are stored), and theses two files exist for both AppxBundle and Appx.

The tree of an AppxBundle look like :
<ul>
	<li>/ (root of the AppxBundle)</li>
	<li>/AppManifest</li>
	<li>/AppManifest/AppxManifest.xml</li>
	<li>/BlockMaps.xml</li>
	<li>/ARM.appx</li>
	<li>/x86.appx</li>
	<li>/x64.appx</li>
	<li>(all the architectures are in the bundle, I removed the non essential files)</li>
</ul>
For Sideloading and changing the certificate to a good one here are the steps we need :
<ol>
	<li>Unbundle appxbundle</li>
	<li>Unpack all the appx</li>
	<li>Modify the AppManifest.xml, for all of them. We need to change the "Publisher" attributes in the XML.</li>
	<li>Modify the BlockMaps.xml, for all of them. We need to remove the Hash and size of the modified AppManifest.xml.</li>
	<li>Pack the appx and sign it</li>
	<li>Create the bundle and sign it</li>
	<li>Create a powershell script to install the dependencies and the bundle.</li>
	<li>Find a way to launch the Powershell Script on all our client</li>
</ol>
The part 2 is coming, soon...

Other Parts  :
<ul>
	<li><a href="http://etienne.deneuve.xyz/2016/01/11/side-loading-application-without-sccm-part-2/" target="_blank">Part 2</a></li>
	<li><a href="http://etienne.deneuve.xyz/2016/01/11/side-loading-application-without-sccm-part-3/" target="_blank">Part 3</a></li>
	<li>The full script is on my (New) Git : <a href="https://github.com/EtienneDeneuve/Powershell">Go to Git !</a></li>
</ul>