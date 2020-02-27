---
ID: 391
title: 'Terraform: Providers à connaître, DataSource à redécouvrir !'
author: etienne.deneuve
post_excerpt: ""
layout: layouts/post-sidebar.njk
mySlug: terraform-providers-datasource
permalink: "{{ page.date | date: '%Y/%m/%d' }}/{{ mySlug }}/index.html"
published: true
date: 2018-04-06 16:30:52
---
# Terraform

## Avant de "coder"

Je ne vous présente pas à nouveau Terraform, je l'ai déjà fait [ici](https://etienne.deneuve.xyz/2017/10/01/microsoft-experience-17-infrastructure-code-modelisez-et-provisionnez-vos-services-azure-avec-terraform-et-packer/), [ici](https://stanislas.io/2017/01/24/infrastructure-dans-azure-c-est-comme-du-lego-slides-et-ressources-utiles/) ou sinon sur mon [github](https://github.com/etiennedeneuve/Azure/tree/master/Terraform). Je vous invite également à regarder la [vidéo](https://stanislas.io/2017/09/18/video-infrastructure-as-code-dans-azure-avec-terraform/) de Stanislas Quastana.
Si vous êtes des partenaires Microsoft, vous pouvez (devez) suivre les webinars [Azure Lab Experiences](http://aka.ms/azure-lab) de [Mickael Debois](https://www.linkedin.com/in/oseborn/) et de [Florent Chambon](https://www.linkedin.com/in/florent-chambon/).
L'objet de cet article est de vous montrer quelques points intéressants qu'il est bon de connaître !

## Providers

Terraform repose sur des providers que nous pouvons déclarer tel que :

```
provider &quot;nomduprovider&quot; {

}
```

J'en ai trouvé qui sont plutôt sympas  fournis par Hashicorp:

- [Random](https://www.terraform.io/docs/providers/random/index.html)
- [Template](https://www.terraform.io/docs/providers/template/index.html)

ou encore ceux de la communauté que je ne détaillerai pas, mais à  noter la présence d'un provider Active Directory, Google Calendar, Let's encrypt... la liste complète [ici](https://www.terraform.io/docs/providers/type/community-index.html).

### Random

Voici un exemple pour le provider random :

```
provider &quot;random&quot; {}

resource &quot;random_pet&quot; &quot;demo_pet&quot; {
prefix = &quot;myad&quot;
separator = &quot;.&quot;
}

resource &quot;random_id&quot; &quot;demo_id&quot; {
prefix = &quot;web&quot;
byte_length = 8
}

output &quot;demo&quot; {
value = &quot;${random_pet.demo_pet.id}&quot;
}

output &quot;demo-id&quot; {
value = &quot;${random_id.demo_id.dec}&quot;
}
```

Recopiez cet exemple puis exécutez la commande ``terraform init`` :

```
PS C:\Users\etien\Documents\it\blog&gt; terraform init

Initializing provider plugins...
- Checking for available provider plugins on https://releases.hashicorp.com...
- Downloading plugin for provider "random" (1.2.0)...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.random: version = "~&gt; 1.2"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Une fois que l'init est fait, nous sommes prêts ! lancez donc ``terraform apply`` :

```
PS C:\Users\etien\Documents\it\blog&gt; terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
+ create

Terraform will perform the following actions:

+ random_id.demo_id
id:
b64:
b64_std:
b64_url:
byte_length: &quot;8&quot;
dec:
hex:
prefix: &quot;web&quot;

+ random_pet.demo_pet
id:
length: &quot;2&quot;
prefix: &quot;myad&quot;
separator: &quot;.&quot;

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
Terraform will perform the actions described above.
Only &#039;yes&#039; will be accepted to approve.

Enter a value: yes

random_pet.demo_pet: Creating...
length: &quot;&quot; =&gt; &quot;2&quot;
prefix: &quot;&quot; =&gt; &quot;myad&quot;
separator: &quot;&quot; =&gt; &quot;.&quot;
random_id.demo_id: Creating...
b64: &quot;&quot; =&gt; &quot;&quot;
b64_std: &quot;&quot; =&gt; &quot;&quot;
b64_url: &quot;&quot; =&gt; &quot;&quot;
byte_length: &quot;&quot; =&gt; &quot;8&quot;
dec: &quot;&quot; =&gt; &quot;&quot;
hex: &quot;&quot; =&gt; &quot;&quot;
prefix: &quot;&quot; =&gt; &quot;web&quot;
random_pet.demo_pet: Creation complete after 0s (ID: myad.quiet.filly)
random_id.demo_id: Creation complete after 0s (ID: aZLeiFCrOno)

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

demo = myad.quiet.filly
demo-id = web7607387397632506490
```

Voilà, grâce à ce provider, plus besoin de chercher des noms pour vos tests et vos déploiments de cattle !

### Template

Pour le provider template, reprenez votre fichier et ajoutez les éléments suivants :

```
provider &quot;random&quot; {}

resource &quot;random_pet&quot; &quot;demo_pet&quot; {
prefix = &quot;myad&quot;
separator = &quot;.&quot;
}

resource &quot;random_id&quot; &quot;demo_id&quot; {
prefix = &quot;web&quot;
byte_length = 4
}

output &quot;demo&quot; {
value = &quot;${random_pet.demo_pet.id}&quot;
}

output &quot;demo-id&quot; {
value = &quot;${random_id.demo_id.dec}&quot;
}

provider &quot;template&quot; {}

data &quot;template_file&quot; &quot;demo&quot; {
template = &lt;&lt;-EOF #/bin/shell hostname $${hostname} EOF vars { hostname = &quot;${random_pet.demo_pet.id}&quot; } } output &quot;demo-template&quot; { value = &quot;${data.template_file.demo.*.rendered}&quot; } ``` puis faites votre init, comme nous avons un nouveau provider, puis votre apply : ``` PS C:\Users\etien\Documents\it\blog&gt; terraform apply
random_pet.demo_pet: Refreshing state... (ID: myad.quiet.filly)
random_id.demo_id: Refreshing state... (ID: ksLYNA)
data.template_file.demo: Refreshing state...

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

demo = myad.quiet.filly
demo-id = web2462242868
demo-template = [
#/bin/shell

hostname myad.quiet.filly

]
```

Ce provider est assez pratique pour générer des user_data ou des scripts customisés à chaque lancement en fonction des variables du contexte.

## DataSources

Comme nous venons de le voir, à moins d&#039;avoir fait un copier coller sans lire le code, dans Terraform nous avions plusieurs type dont &quot;provider&quot;, &quot;resource&quot; ou encore &quot;output&quot; et &quot;variable&quot;, mais aussi &quot;data&quot;. Ce type est assez intéressant puisqu&#039;il permet d&#039;interroger dynamiquement le provider.
Admettons, vous avez une belle infra réseau dans Azure et vous souhaitez ajouter une belle machine dans ce beau réseau, avant ce n&#039;était pas super pratique, il fallait jouer avec les imports, au risque de supprimer le vnet (par erreur, un vendredi soir vers 17h, au hazard) lors de l&#039;invocation de la meilleure commande de terraform (destroy).

Voici un petit exemple de l&#039;utilisation de ces datasources :

```
provider "azurerm" {

}

data "azurerm_virtual_network" "test" {
name = "production"
resource_group_name = "networking"
}

output "virtual_network_id" {
value = "${data.azurerm_virtual_network.test.id}"
}
```

&gt; Vous noterez surement que mon provider azurerm est vide, non je n&#039;ai pas effacé mes infos, mais je vais vous donner deux solutions pour ne pas les oublier avant de faire un commit sur un github public avec votre SPN en clair pour tous.
&gt;
&gt; - Solution 1 :
&gt; Installez Azure Cli 2.0
&gt; - Solution 2 :
&gt; Ajouter a vos variables d&#039;environnement les elements :
&gt;
&gt; * ARM_SUBSCRIPTION_ID,
&gt; * ARM_CLIENT_ID,
&gt; * ARM_CLIENT_SECRET,
&gt; * ARM_TENANT_ID
&gt;
&gt;Si vous utilisez Powershell, n&#039;oubliez pas que vous pouvez les créer de cette façon :
&gt;```
&gt;Set-Item env:\ARM_SUBSCRIPTION_ID -Value votrevalue
&gt;```

# Pour conclure

J'èspere que ça vous sera utile :)

Si vous souhaitez en savoir plus, je présente une session au Global Azure Bootcamp le 21 avril à l'école 42 répondant au titre de : "(Terraform + Packer + Ansible) + Azure = &lt;3 ?"