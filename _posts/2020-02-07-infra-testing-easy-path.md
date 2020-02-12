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
# IaC & Tests

<!-- vscode-markdown-toc -->
* 1. [Overview](#Overview)
* 2. [Workstation preparation](#Workstationpreparation)
	* 2.1. [Install Powershell and modules](#InstallPowershellandmodules)
		* 2.1.1. [Powershell 6](#Powershell6)
		* 2.1.2. [Modules](#Modules)
	* 2.2. [Visual Studio Code](#VisualStudioCode)
		* 2.2.1. [Extensions installation](#Extensionsinstallation)
* 3. [Pester, Gherkin and testing](#PesterGherkinandtesting)
* 4. [Let's play](#Letsplay)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

##  1. <a name='Overview'></a>Overview

I was looking for the easiest way to test Azure infrastructure.
I took a look into great project like Molecule, Terratest and so on.
Even if they are very cool and powerful, I wanted to find something easier (or lazier).

I'm actually using pester for testing purpose, so I'm managing to use pester to test Azure infrastructure,
 using the Gherkin syntax and I'll show you, how you can do that.

##  2. <a name='Workstationpreparation'></a>Workstation preparation

I work on 3 kind of platforms : macOs, Ubuntu and Windows 10, and my tests are working on all of them.

You will need to install :

1. PowerShell (6.2.3 at least)
   1. Powershell Az module
   2. Pester (at least 4.9)
2. VS Code
   1. [Gherkin plugin](https://marketplace.visualstudio.com/items?itemName=alexkrechik.cucumberautocomplete)
   2. [PowerShell](https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell)

###  2.1. <a name='InstallPowershellandmodules'></a>Install Powershell and modules

####  2.1.1. <a name='Powershell6'></a>Powershell 6

##### Manual installation

First, download the suitable version of PowerShell for your system on GitHub. Yes, if you miss it, that is now Open Source.  
<https://github.com/PowerShell/PowerShell/releases>

##### Package Manager

###### macOS

If you use macOs, [Brew](https://brew.sh/) can manage it for you:

```sh
brew install powershell
```

###### Ubuntu

On Ubuntu, Snap can do the job also :

```zsh
sudo snap install powershell --classic
```

###### Windows

On Windows, Chocolatey is working well:  

```powershell
choco install pwsh
```

####  2.1.2. <a name='Modules'></a>Modules

Installing module in PowerShell is quite easy. You only need to type the following :

```Powershell
Install-PackageProvider Nuget -ForceBootstrap -Force
Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
Update-Module
Install-Module -Name Pester -Force -SkipPublisherCheck
Install-Module -Name Az -force
```

> Some commands may not be useful on your setup, but this way ensures that you have the latest version of each module

###  2.2. <a name='VisualStudioCode'></a>Visual Studio Code

You can install Vs Code manually or by using a Package Manager using `choco`, `snap` or `brew`. I'll not detail this part, that's the same as in [Powershell 6](#Powershell6).

####  2.2.1. <a name='Extensionsinstallation'></a>Extensions installation

To install the extension, you can use the GUI of Vs Code or the cmdline. For Gui, take a look at the doc [here](https://code.visualstudio.com/docs/editor/extension-gallery).

For the cmdline :

```shell
code --install-extension alexkrechik.cucumberautocomplete
code --install-extension ms-vscode.powershell
```

> on Linux and macOs, you'll need to add the following in your profile, adapt the path to Vs Code to yours :
>
> __bash__:
>
> ```bash
> cat << EOF >> ~/.bash_profile
> # Add Visual Studio Code (code)
> export PATH="\$PATH:/Applications/Visual Studio Code.app/Contents/Resources/app/bin"
> EOF
> ```

> __zsh__:

> ```
> cat << EOF >> ~/.zsh_profile
> # Add Visual Studio Code (code)
> export PATH="\$PATH:/Applications/Visual Studio Code.app/Contents/Resources/app/bin"
> EOF
> ```

##  3. <a name='PesterGherkinandtesting'></a>Pester, Gherkin and testing

First of all, you will need some details on Pester, Gherkin and testing like I think they can be useful and easy as possible.

[Pester](https://github.com/pester/Pester), is a world community framework for testing in Powershell. So, if you know a bit in PowerShell, you won't be lost at all. This module add some new functions to do the magic of testing.

[Gherkin](https://cucumber.io/docs/gherkin/reference/), is a syntax for BDD (Behavior-Driven Development) testing. To explain a bit, for me, in our infrastructure testing purpose, is to have some human readable scenario to show to management/customers.

Fortunately, Pester support the Gherkin syntax using the `Invoke-Gherkin` cmdlet and do a bit of magic for us.

##  4. <a name='Letsplay'></a>Let's play

We need to create our workspace, so do it like that (Using PowerShell's Shell):

```Powershell
Set-Location ~
New-Item ./Documents/GherkinTests -type Directory
code ./Documents/GherkinTests
```

Create few file :

- etienne_exo.feature <-- This is the Gherkin File
- etienne_exo.steps.ps1 <-- This is our magic

In the Feature File write the following :

```Gherkin
# GherkinTests/etienne_exo.feature
Feature: Validate Azure Deployment

  Scenario: We should have some subscriptions to work
    Given we list the subscriptions using powershell
    Then we should be able to have at least one
```

```Powershell
# GherkinTests/etienne_exo.steps.ps1
Given "we list the subscriptions using powershell"{
  Get-AzSubscription | Should -not -throw
}

Then "we should be able to have at least one" {
  (Get-AzSubscription).Count | Should -Not -BeNullOrEmpty
}
```

Now, let's run that to check if the test are working or not :

```PowerShell
❯ Invoke-Gherkin
Pester v4.10.0
Executing all tests in '/home/etienne/Documents/GherkinTests'

Feature: Validate Azure Deployment

  Scenario: We should have some subscriptions to work
    [+] Given we list the subscriptions using powershell 6.17s
    [+] Then we should be able to have at least one 5.01s
Tests completed in 11.39s
Tests Passed: 2, Failed: 0, Skipped: 0, Pending: 0, Inconclusive: 0
```

Ok, tests are working, and we have something to show to customers/managment.

> Wait, can't we do better ? Are we lazy enough ?
> Of course, not, let's wait for the next blog post.
