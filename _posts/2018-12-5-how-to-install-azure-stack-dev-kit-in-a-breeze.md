---
ID: 473
post_title: >
  how to install azure stack dev kit in a breeze
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2018/07/25/install-asdk-in-a-breeze/
published: true
post_date: 2018-12-05 12:35:17
---
# Azure Stack

Azure Stack is an on premise appliance made by Microsoft. The main goal of Azure Stack isn't to have a new Hypervisor like vSphere or Hyper-V, but to have real cloud on prem. If you want to deploy only Virtual Machines, you aren't on the good path.

This article is summary for installing and configuring an Azure Stack with a lot of PowerShell coming from GitHub. With all the scripts you may have a running Azure Stack.

## Hardware prerequisites

You need a server with at least the following configuration :

| Component                                           | Minimum                                                                                        | Recommended                                                                                             |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Disk drives: Operating System                       | 1 OS disk with minimum of 200 GB available for system partition (SSD or HDD)                   | 1 OS disk with minimum of 200 GB available for system partition (SSD or HDD)                            |
| Disk drives: General development kit data* |	4 disks.  Each disk provides a minimum of 140 GB of capacity (SSD or HDD). All available disks are used. | 4 disks. Each disk provides a minimum of 250 GB of capacity (SSD or HDD). All available disks are used. |
| Compute: CPU                                        | Dual-Socket: 12 Physical Cores (total)                                                         | Dual-Socket: 16 Physical Cores (total)                                                                  |
| Compute: Memory                                     | 96 GB RAM                                                                                      | 128 GB RAM (This is the minimum to support PaaS resource providers.)                                    |
| Compute: BIOS                                       | Hyper-V Enabled (with SLAT support)                                                            | Hyper-V Enabled (with SLAT support)                                                                     |
| Network: NIC                                        | Windows Server 2012 R2 Certification required for NIC; no specialized features required        | Windows Server 2012 R2 Certification required for NIC; no specialized features required                 |
| HW logo certification                               | Certified for Windows Server 2012 R2                                                           | Certified for Windows Server 2016                                                                       |

Source : [docs.microsoft.com](https://docs.microsoft.com/en-us/azure/azure-stack/asdk/asdk-deploy-considerations#hardware)

You can deploy Azure Stack Developpement kit in Azure using a E16s v3

## Windows Installation

You need to install a Windows Server 2016 with a full disk (no partition). As the installer will boot on a VHD on top of the system root, if the second drive isn't big enough the installer will not thrown an error and you will get a bad installation of Azure Stack.

You need to download the following files :

- Script to check the requirements (not mandatory, but useful) [link](https://gallery.technet.microsoft.com/Deployment-Checker-for-50e0f51b)
- A Windows Server 2016 Eval Iso [lien](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016)
- Azure Stack Development Kit [lien](https://azure.microsoft.com/fr-fr/overview/azure-stack/development-kit/?v=try)

### Installation

#### Phase 1

1. Extract the ADSK using AzureStackDevelopmentKit.exe
1. Copy "CloudBuilder.vhdx" on the root of you root (c:)

#### Phase 2

In a Shell Powershell as Administrator :

```powershell
# Variables
$Uri = 'https://raw.githubusercontent.com/Azure/AzureStack-Tools/master/Deployment/asdk-installer.ps1'
$LocalPath = 'C:\AzureStack_Installer'
# Create folder
New-Item $LocalPath -Type directory
# Enforce usage of TLSv1.2 to download from GitHub
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
# Download file
Invoke-WebRequest $uri -OutFile ($LocalPath + '\' + 'asdk-installer.ps1')
cd $LocalPath
.\asdk-installer.ps1
```

Follow "Prepare Environnement" 

1. Browse to your "cloudbuilder.vhdx" from phase 1 (c:\CloudBuilder.vhdx), without ticking the Add Drivers (if your hardware is fully supported as a Dell ASDK Ready Node for example)
1. Submit your administrator's credentials of the local machine.
1. You can add a Computer Name for the Azure Stack, if you don't it will be generated. You must choose something different than AzureStack
1. If you have a Dhcp, you don't need to add a Static Configuration, otherwise, add a static configuration instead (with DNS and Gateway)
1. Reboot

#### Phase 3

While the host is rebooting, if the Windows bootmanager ask to choose between Azure Stack and Windows Server 2016, choose Azure Stack.

Login in the Azure Stack, take care of the keyboard layout, by default it's a QWERTY.

In a PowerShell Shell, as Administrator launch the ASDK deployment. (Not in a Powershell ISE)

```Powershell
cd C:\CloudDeployment\Setup     
$adminpass = Get-Credential Administrator  -Message "Please provide the password for Local Administrator"
$aadcred = Get-Credential "svc_azure_stack_installer@yourazuread.onmicrosoft.com" -Message "Please provide the password for Azure AD"
.\InstallAzureStackPOC.ps1 -AdminPassword $adminpass.Password `
    -InfraAzureDirectoryTenantName "yourazuread.onmicrosoft.com" `
    -TimeServer "13.79.239.69"`
    -InfraAzureDirectoryTenantAdminCredential $aadcred`
    -DNSForwarder "8.8.8.8" `
    -NatIPv4Address "10.0.0.9" `
    -NatIPv4DefaultGateway "10.0.0.14" `
    -NatIPv4Subnet "10.0.0.0/28"
```

Warning: This process take few hours (between 10 and 15)

> You can change parameter in the script, but double check they are correct. If you have trouble, you may need to restart the whole process.
> The nat parameters must be changed to reflect your network.

#### Phase 4

After the process is complete, you have now a cool ASDK, with nothing in it. You need to deploy services as a Cloud Operator

1. Login with azurestack\azurestackadmin on the ASDK host
1. Get the path of your Windows Server 2016, mine is 'D:\ISO'
1. Open a new Powershell as Administator :

```Powershell
# Create directory on the root drive.
New-Item -ItemType Directory -Force -Path "C:\ConfigASDK"
Set-Location "C:\ConfigASDK"

# Download the ConfigASDK Script.
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Invoke-Webrequest http://bit.ly/configasdk -UseBasicParsing -OutFile ConfigASDK.ps1

.\ConfigASDK.ps1 -azureDirectoryTenantName "yourazuread.onmicrosoft.com" -authenticationType AzureAD `
    -downloadPath "D:\ASDKfiles" -ISOPath "D:\ISO\WS2016EVAL.iso" -azureStackAdminPwd 'Azure Stack Administrator password' `
    -VMpwd 'a new password' -azureAdUsername "svc_azure_stack_installer@yourazuread.onmicrosoft.com" -azureAdPwd 'Azure Ad password' `
    -registerASDK -useAzureCredsForRegistration -azureRegSubId "Id de la subscription Azure"
```

#### Phase 5

Open a Browser on your ASDK Host and go to ``https://adminportal.local.azurestack.external`` or the "public" endpoint ``https://portal.local.azurestack.external`` and enjoy.