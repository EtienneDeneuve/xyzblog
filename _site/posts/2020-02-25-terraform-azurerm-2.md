---
layout: layouts/post-sidebar.njk
title: 'Terraform AzureRM 2.0 Provider'
author: etienne.deneuve
sidebar: infra-test
eleventyNavigation:
  key: seven-styles
  title: 'Terraform AzureRM 2.0 Provider'
  parent: posts
  order: 1
tags:
  - Azure
  - PowerShell
  - Azure Devops
  - Cloud
  - Tests
---

# New Azure RM Provider 2.0

This article will be updated every times I found something not yet documented on the 2.0 Azure RM provider for Terraform.

## New mandatory

In the past, the provider was optional, today, it is *mandatory* !

You need to write this part :

``` hcl
provider "azurerm" {
  version = "=2.0.0"
  features {}
}
```

If you don't add the `features {}` your plan and deployment will fail.

As today, the features block is the new way to activate some features :

> features block supports the following:
>
> - key_vault - (Optional) A key_vault block as defined below.
>
> - virtual_machine - (Optional) A virtual_machine block as > defined below.
>
> - virtual_machine_scale_set - (Optional) A > virtual_machine_scale_set block as defined below.
