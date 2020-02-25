---
ID: 500
post_title: Terraform AzureRM 2.0 Provider
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2020/02/25/terraform-azurerm-2-0-provider/
published: true
post_date: 2020-02-25 10:29:24
---
<h1>New Azure RM Provider 2.0</h1>

This article will be updated every times I found something not yet documented on the 2.0 Azure RM provider for Terraform.

<h2>New mandatory</h2>

In the past, the provider was optional, today, it is <em>mandatory</em> !

You need to write this part :

<pre><code class="language-HCL">provider "azurerm" {
  version = "=2.0.0"
  features {}
}
</code></pre>

If you don't add the <code>features {}</code> your plan and deployment will fail.

As today, the features block is the new way to activate some features :

<blockquote>features block supports the following:
<ul>
    <li>key_vault - (Optional) A key_vault block as defined below.</li>
    <li>virtual_machine - (Optional) A virtual_machine block as &gt; defined below.</li>
    <li>virtual_machine_scale_set - (Optional) A &gt; virtual_machine_scale_set block as defined below.</li>
</ul>
</blockquote>