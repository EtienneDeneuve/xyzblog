---
ID: 500
post_title: Terraform AzureRM 2.0 Provider
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  terraform_azurerm_2
published: true
post_date: 2020-02-25 10:00:00
---

# New Azure RM Provider 2.0

This article will be updated every times I found something not yet documented on the 2.0 Azure RM provider for Terraform.

## New mandatory

In the past, the provider was optional, today, it is *mandatory* !

You need to write this part :

```HCL
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
