---
layout: layouts/post-sidebar.njk
title: 'IaC & Tests'
summary: "I was looking for the easiest way to test Azure infrastructure.
I wanted to find something easier (or lazier)."
# hero: /images/posts/chromeextensions.png
# thumb: /images/posts/chromeextensions_tn.png
sidebar: infra-test
eleventyNavigation:
  key: seven-styles
  title: 'IaC & Tests'
  parent: posts
  order: 1
tags:
  - Azure
  - PowerShell
  - Azure Devops
  - Cloud
  - Tests
---

# IaC & Tests

## Overview

I was looking for the easiest way to test Azure infrastructure.
I take a look into great project like Molecule, Terratest and so on.
Even if they are very cool and powerful, I wanted to find something easier (or lazier).

I already use pester for testing purpose, so I manage to use pester for testing Azure infrastructure,
 using the Gherkin syntax and I'll show you, how you can do that.

##  Workstation preparation

I work on 3 kind of platform : macOs, Ubuntu and Windows 10, and my tests are working on all of them.

You will need to install :

1. PowerShell (6.2.3 at least)
   1. Powershell Az module
   2. Pester (at least 4.9)
2. VS Code
   1. [Gherkin plugin](https://marketplace.visualstudio.com/items?itemName=alexkrechik.cucumberautocomplete)
   2. [PowerShell](https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell)

### Install Powershell and modules

#### Powershell 6

##### Manual installation

First, download the suitable version of powershell for your system on GitHub. Yes, if you miss it, that is now fully open source.  
<https://github.com/PowerShell/PowerShell/releases>

##### Package Manager

###### macOS

If you use macOs, [brew](https://brew.sh/) can manage it  for you:

```
brew install powershell
```

###### Ubuntu

On Ubuntu, snap can do the job also :

```
sudo snap install powershell --classic
```

###### Windows

On Windows, Chocolatey is working well:  

```
choco install pwsh
```

#### Modules

Installing module in PowerShell is quite easy. You only need to type the following :

```
Install-PackageProvider Nuget -ForceBootstrap -Force
Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
Update-Module
Install-Module -Name Pester -Force -SkipPublisherCheck
Install-Module -Name Az -force
```

> Some command may not be useful on your setup, but this way ensure that you have the latest version of each module

### Visual Studio Code

You can install Vs Code manually or by using a Package Manager using choco, snap or brew. I'll not detail this part.

####  Extensions installation

To install the extension, you can use the GUI of Vs Code or the cmdline. For Gui, take a look at the doc [here](https://code.visualstudio.com/docs/editor/extension-gallery).

For the cmdline :

```
code --install-extension alexkrechik.cucumberautocomplete
code --install-extension ms-vscode.powershell
```

> on Linux and macOs, you'll need to add the following in your profile, adapt the path to Vs Code to yours :
>
> __bash__:
>
> ```
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

## Create the first tests

### Pester, Gherkin and testing

First of all, you will need some detail on Pester, Gherkin and testing like I think they can be useful and easy as possible.

[Pester](https://github.com/pester/Pester), is a world community framework for testing in Powershell. So, if you know a bit in PowerShell, you won't be lost at all. This module add some new function to do the magic of testing.

[Gherkin](https://cucumber.io/docs/gherkin/reference/), is a syntax for BDD (Behavior-Driven Development) testing. To explain a bit, for me, in our infrastructure testing purpose, is to have some human readable scenario to show to management/customers.

Fortunately, Pester support the Gherkin syntax using the `Invoke-Gherkin` cmdlet and do a bit of magic for us.

### Let's play

We need to create our workspace, so do it like that (Using PowerShell's Shell):

```
Set-Location ~
New-Item ./Documents/GherkinTests -type Directory
code ./Documents/GherkinTests
```

Create few file :

- etienne_exo.feature <-- This is the Gherkin File
- etienne_exo.steps.ps1 <-- This is our magic

In the Feature File write the following :

```
# GherkinTests/etienne_exo.feature
Feature: Validate Azure Deployment

  Scenario: We should have some subscriptions to work
    Given we list the subscriptions using powershell
    Then we should be able to have at least one
```

```
# GherkinTests/etienne_exo.steps.ps1
Given "we list the subscriptions using powershell"{
  Get-AzSubscription | Should -not -throw
}

Then "we should be able to have at least one" {
  (Get-AzSubscription).Count | Should -Not -BeNullOrEmpty
}
```

Now, let's run that to check if the test are working or not :

```
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

> Wait, can't we do nothing better ? Are we lazy enough ?
> Of course, not, let's wait for the next blog part.

## Azure Devops
### Azure Devops CLI

Get a Personal Access Token here : `https://dev.azure.com/<YourOrganisation>/_usersSettings/tokens`

```
az extension add --name azure-devops
az devops login 
# paste your PAT
az login
# with the sam
az account set -s <YOUR SUB>
az devops configure --defaults 'organization=https://dev.azure.com/etiennedeneuve'
az devops project create --name GherkinTest
# Store the repo git url to configure the repo locally, you will need a SSH Key for that.
$repo=$(az repos list --project GherkinTest --query [].sshUrl -o tsv)
```

### Configure the repo locally

```
cd ~/GherkinTest
git init
git remote add origin $repo
git add .
git commit -m 'inital commit'
git push origin master
```

### Create the pipeline

```
az pipelines create --name "GherkinTest"
```

Answer the questions like :

```
This command is in preview. It may be changed/removed in a future release.
Which template do you want to use for this pipeline?
 [1] Starter pipeline
 [2] Android
 [3] Ant
 [4] ASP.NET
 [5] ASP.NET Core
 [6] .NET Core Function App to Windows on Azure
 [7] ASP.NET Core (.NET Framework)
Please enter a choice [Default choice(1)]: Starter pipeline

Do you want to view/edit the template yaml before proceeding?
Please enter a choice [Default choice(1)]: Continue with generated yaml

Files to be added to your repository (1)
1) azure-pipelines.yml

How do you want to commit the files to the repository?
Please enter a choice [Default choice(1)]: Create a new branch for this commit and start a pull request.

Enter new branch name to create: features/cicd
Checking in file azure-pipelines.yml in the Azure repo c279436a-e2f4-4e01-8f41-a30f660f7515
Created a Pull Request - https://dev.azure.com/etiennedeneuve/6874897d-6d09-412e-b73b-4b2966c04b64/_apis/git/repositories/c279436a-e2f4-4e01-8f41-a30f660f7515/pullRequests/1
Successfully created a pipeline with Name: GherkinTest, Id: 25.
{
  "agentSpecification": null,
  "buildNumber": "20200214.1",
  "buildNumberRevision": 1,
  "controller": null
}
```

###  Create the Service Endpoint for Azure RM Subscription

First we need to create a SPN in Azure AD :

```
az ad sp create-for-rbac --name AzureDevops
Changing "AzureDevops" to a valid URI of "http://AzureDevops", which is the required format used for service principal names
Creating a role assignment under the scope of "/subscriptions/1417c648-XXXX"
 
{
  "appId": "41176fe8-XXXXX",
  "displayName": "AzureDevops",
  "name": "http://AzureDevops",
  "password": "7eaeb380-XXXXX",
  "tenant": "06329ce4-XXXX"
}
```

Take note of the appId, Name, Password and Tenant.

Now, list your subscriptions :

```
az account show
{
  "environmentName": "AzureCloud",
  "id": "1417c648-XXXXX",
  "isDefault": true,
  "name": "Microsoft XXXX",
  "state": "Enabled",
  "tenantId": "06329ce4-XXXX",
  "user": {
    "name": "etienne@deneuve.xyz",
    "type": "user"
  }
}
```

Take note of the Id, Name, and Tenant.

And then create the service endpoint in Azure DevOps :

```
 az devops service-endpoint azurerm create --name 'Azure MVP' `
>> --azure-rm-tenant-id "YourTenantId" `
>>     --azure-rm-service-principal-id "AppID" `
>>     --azure-rm-subscription-id "SubID" `
>>     --azure-rm-subscription-name "Name of the Sub"
Azure RM service principal key: "Password"
Confirm Azure RM service principal key: "Password"
{
  "administratorsGroup": null,
   < shortened >
  "type": "azurerm",
  "url": "https://management.azure.com/"
}
```

### Configure the pipeline

Now, checkout to the newly created branch :

```
# change branch
git checkout features/cicd
# get latests info from remote
git pull
# open code in the current folder
code .
```

Add the following snippet into `azure-pipelines.yml` :

```
# ./azure-pipelines.yml
# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- master

pool:
  vmImage: 'windows-latest'

steps:
- task: AzurePowerShell@5
  inputs:
    azurePowerShellVersion: LatestVersion
    azureSubscription: 'Azure MVP'
    Inline: |
      Install-Module -Name Pester -Force
    ScriptType: InlineScript
    pwsh: true
    workingDirectory: $(Build.Repository.LocalPath)
  displayName: 'Install Pester'

- task: AzurePowerShell@5
  inputs:
    azurePowerShellVersion: LatestVersion
    azureSubscription: 'Azure MVP'
    Inline: |
      Invoke-Gherkin -OutputFile result.xml -OutputFormat NUnitXml
    ScriptType: InlineScript
    pwsh: true
    workingDirectory: $(Build.Repository.LocalPath)
  displayName: 'Launch Test'

- task: PublishTestResults@2
  inputs:
    buildConfiguration: Azure
    buildPlatform: Azure
    publishRunAttachments: true
    testResultsFiles: result.xml
    testResultsFormat: NUnit
    testRunTitle: ValidateAzure
```

Commit the file :

```
git add azure-pipelines.yaml
git commit -m 'feat: add cicd for tests'
git push origin features/cicd
```

Go in your Azure Devops project, select your new pipeline, and go in Tests, you should have something like my public repo :

[Azure Devops](https://dev.azure.com/etiennedeneuve/gherkintest/_build/results?buildId=357&view=ms.vss-test-web.build-test-results-tab)

*Et Voila!*

## Working with the tests

Now, if we want to deploy something in our new infra, we will need to write the tests before deploying the resources themselves.

### Adding some tests

In this scenario, we want to deploy a VM, in our subscription. Let's do a quick list of what we should have after a successful deployment :

- 1 Subscription
- 1 Resource Group
- 1 Virtual Network
- 1 Subnet
- 1 Network Interface
- 1 Disk
- 1 VM
- Optionally: 1 Public IP

With our list, we can now write the test we want, in the feature file:

```
# GherkinTests/etienne_exo.feature
Feature: Validate Azure Deployment

  Scenario: Someone start a new vm deployment
    Given someone start a new vm deployment in the subscription
    When the deployment is completed we should have 1 resource group
    Then we should have 1 Virtual Network
    And 1 Subnet
    And 1 Network interface
    And 1 Disk
    And 1 VM
    And 1 Public IP
```

Now, in our steps.ps1 file we will update it:

```
# GherkinTests/etienne_exo.steps.ps1
Given "someone start a new vm deployment in the subscription"{
  Get-AzSubscription | Should -not -throw
}

When "the deployment is completed we should have 1 resource group" {
  Get-AzResourceGroup -name RG-GherkinTests | Should -exists
}

Then "We should have 1 Virtual Network"{

}
And "1 Subnet"{

}
And "1 Network interface"{

}
And "1 Disk"{

}
And "1 VM"{

}
And "1 Public IP"{

}
```