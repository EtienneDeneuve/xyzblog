---
ID: 498
post_title: Testing your infra my easy way
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2020/02/07/infra-testing-easy-way/
published: true
post_date: 2020-02-07 16:00:00
---
<h1>IaC &amp; Tests</h1>

<!-- vscode-markdown-toc -->
* 1. <a href="#Overview">Overview</a>
* 2. <a href="#Workstationpreparation">Workstation preparation</a>
* 2.1. <a href="#InstallPowershellandmodules">Install Powershell and modules</a>
* 2.1.1. <a href="#Powershell6">Powershell 6</a>
* 2.1.2. <a href="#Modules">Modules</a>
* 2.2. <a href="#VisualStudioCode">Visual Studio Code</a>
* 2.2.1. <a href="#Extensionsinstallation">Extensions installation</a>
* 3. <a href="#PesterGherkinandtesting">Pester, Gherkin and testing</a>
* 4. <a href="#Letsplay">Let's play</a>

<!-- vscode-markdown-toc-config
    numbering=true
    autoSave=true
    /vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

<h2>1. <a name="Overview"></a>Overview</h2>

I was looking for the easiest way to test Azure infrastructure.
I take a look into great project like Molecule, Terratest and so on.
Even if they are very cool and powerful, I wanted to find something easier (or lazier).

I already use pester for testing purpose, so I manage to use pester for testing Azure infrastructure,
using the Gherkin syntax and I'll show you, how you can do that.

<h2>2. <a name="Workstationpreparation"></a>Workstation preparation</h2>

I work on 3 kind of platform : macOs, Ubuntu and Windows 10, and my tests are working on all of them.

You will need to install :

<ol>
    <li>PowerShell (6.2.3 at least)
<ol>
    <li>Powershell Az module</li>
    <li>Pester (at least 4.9)</li>
</ol>
</li>
    <li>VS Code
<ol>
    <li><a href="https://marketplace.visualstudio.com/items?itemName=alexkrechik.cucumberautocomplete">Gherkin plugin</a></li>
    <li><a href="https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell">PowerShell</a></li>
</ol>
</li>
</ol>

<h3>2.1. <a name="InstallPowershellandmodules"></a>Install Powershell and modules</h3>

<h4>2.1.1. <a name="Powershell6"></a>Powershell 6</h4>

<h5>Manual installation</h5>

First, download the suitable version of powershell for your system on GitHub. Yes, if you miss it, that is now fully open source.
<a href="https://github.com/PowerShell/PowerShell/releases">https://github.com/PowerShell/PowerShell/releases</a>

<h5>Package Manager</h5>

<h6>macOS</h6>

If you use macOs, <a href="https://brew.sh/">brew</a> can manage it  for you:

<pre><code class="language-sh">brew install powershell
</code></pre>

<h6>Ubuntu</h6>

On Ubuntu, snap can do the job also :

<pre><code class="language-zsh">sudo snap install powershell --classic
</code></pre>

<h6>Windows</h6>

On Windows, Chocolatey is working well:

<pre><code class="language-powershell">choco install pwsh
</code></pre>

<h4>2.1.2. <a name="Modules"></a>Modules</h4>

Installing module in PowerShell is quite easy. You only need to type the following :

<pre><code class="language-Powershell">Install-PackageProvider Nuget -ForceBootstrap -Force
Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
Update-Module
Install-Module -Name Pester -Force -SkipPublisherCheck
Install-Module -Name Az -force
</code></pre>

<blockquote>
  Some command may not be useful on your setup, but this way ensure that you have the latest version of each module</blockquote>

<h3>2.2. <a name="VisualStudioCode"></a>Visual Studio Code</h3>

You can install Vs Code manually or by using a Package Manager using choco, snap or brew. I'll not detail this part.

<h4>2.2.1. <a name="Extensionsinstallation"></a>Extensions installation</h4>

To install the extension, you can use the GUI of Vs Code or the cmdline. For Gui, take a look at the doc <a href="https://code.visualstudio.com/docs/editor/extension-gallery">here</a>.

For the cmdline :

<pre><code class="language-shell">code --install-extension alexkrechik.cucumberautocomplete
code --install-extension ms-vscode.powershell
</code></pre>

<blockquote>
  on Linux and macOs, you'll need to add the following in your profile, adapt the path to Vs Code to yours :

<strong>bash</strong>:
<pre><code class="language-bash">cat &lt;&lt; EOF &gt;&gt; ~/.bash_profile
# Add Visual Studio Code (code)
export PATH="$PATH:/Applications/Visual Studio Code.app/Contents/Resources/app/bin"
EOF
</code></pre>
<strong>zsh</strong>:
<pre><code>cat &lt;&lt; EOF &gt;&gt; ~/.zsh_profile
# Add Visual Studio Code (code)
export PATH="$PATH:/Applications/Visual Studio Code.app/Contents/Resources/app/bin"
EOF
</code></pre>
</blockquote>

<h2>3. <a name="PesterGherkinandtesting"></a>Pester, Gherkin and testing</h2>

First of all, you will need some detail on Pester, Gherkin and testing like I think they can be useful and easy as possible.

<a href="https://github.com/pester/Pester">Pester</a>, is a world community framework for testing in Powershell. So, if you know a bit in PowerShell, you won't be lost at all. This module add some new function to do the magic of testing.

<a href="https://cucumber.io/docs/gherkin/reference/">Gherkin</a>, is a syntax for BDD (Behavior-Driven Development) testing. To explain a bit, for me, in our infrastructure testing purpose, is to have some human readable scenario to show to management/customers.

Fortunately, Pester support the Gherkin syntax using the <code>Invoke-Gherkin</code> cmdlet and do a bit of magic for us.

<h2>4. <a name="Letsplay"></a>Let's play</h2>

We need to create our workspace, so do it like that (Using PowerShell's Shell):

<pre><code class="language-Powershell">Set-Location ~
New-Item ./Documents/GherkinTests -type Directory
code ./Documents/GherkinTests
</code></pre>

Create few file :

<ul>
    <li>etienne_exo.feature &lt;-- This is the Gherkin File</li>
    <li>etienne_exo.steps.ps1 &lt;-- This is our magic</li>
</ul>

In the Feature File write the following :

<pre><code class="language-Gherkin"># GherkinTests/etienne_exo.feature
Feature: Validate Azure Deployment

  Scenario: We should have some subscriptions to work
    Given we list the subscriptions using powershell
    Then we should be able to have at least one
</code></pre>

<pre><code class="language-Powershell"># GherkinTests/etienne_exo.steps.ps1
Given "we list the subscriptions using powershell"{
  Get-AzSubscription | Should -not -throw
}

Then "we should be able to have at least one" {
  (Get-AzSubscription).Count | Should -Not -BeNullOrEmpty
}
</code></pre>

Now, let's run that to check if the test are working or not :

<pre><code class="language-PowerShell">❯ Invoke-Gherkin
Pester v4.10.0
Executing all tests in '/home/etienne/Documents/GherkinTests'

Feature: Validate Azure Deployment

  Scenario: We should have some subscriptions to work
    [+] Given we list the subscriptions using powershell 6.17s
    [+] Then we should be able to have at least one 5.01s
Tests completed in 11.39s
Tests Passed: 2, Failed: 0, Skipped: 0, Pending: 0, Inconclusive: 0
</code></pre>

Ok, tests are working, and we have something to show to customers/managment.

<blockquote>
  Wait, can't we do nothing better ? Are we lazy enough ?
Of course, not, let's wait for the next blog post.</blockquote>