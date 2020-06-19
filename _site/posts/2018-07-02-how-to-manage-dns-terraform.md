---
ID: 471
title: >
  Dynamic update of Bind9 server in Azure
  with Terraform !
author: etienne.deneuve
post_excerpt: ""
layout: layouts/post-sidebar.njk
mySlug: how-to-manage-dns-terraform
permalink: "{{ page.date | date: '%Y/%m/%d' }}/{{ mySlug }}/index.html"
published: true
date: 2018-07-02 12:35:17
---
# Terraform your DNS

In this blog post, I post my article posted on Cellenza Blog, translated in English. The french version is <a href="https://blog.cellenza.com/cloud-2/azure/comment-utiliser-hashicorp-terraform-pour-gerer-vos-dns-bind9-dans-azure/">here</a>. 

The purpose of this blog post is to deploy and configure a Bind9 DNS Server in your Azure subscription and make it able to receive DNS Update form Terraform. I'm pretty sure that you will understand why it's so cool ! If not, the goal is to be able to have local DNS that you can manage when you deploy new resources in Azure. Of course, it can work in many other cloud or on premise environment as Bind9 is real standard DNS.

> The way is give is volontary simplified. To make it good for production you need to add a control VM and not expose your DNS directly with a public IP. It's only a proof of concept
<!--more-->


## Terraform DNS Provider

In Terraform plugins, there's a DNS Provider who help to manipulate a DNS correctly setted according to RFC 2136 and 2845. Thoses RFC normalize dns dynamic updates, this is normally used with a DHCP.

## Cloud Init

We will use [Cloud Init](https://cloud-init.io/) for primary setup of our Labs, Cloud Init will be lauchned in the very low boot stage.

## Building the folder structure for the POC

Open Visual Studio Code and create a base folder ``$workdir`` and create the folling files &amp; folders :

```text
- bind
   - main.tf
   - files/cloudconfig.tpl
   - data.tf
   - variables.tf
   - provider.tf
- dns
   - main.tf
   - provider.tf
```

## Deploy Bind9 VM

### Terraform

In the file ``bind/provider.tf``, add this :

```
provider &quot;azurerm&quot; {
version = &quot;1.6.0&quot;
}

provider &quot;template&quot; {
version = &quot;1.0.0&quot;
}
```

In ``bind/data.tf``, add this :

```
data &quot;template_cloudinit_config&quot; &quot;config&quot; {
gzip = true
base64_encode = true

part {
content_type = &quot;text/cloud-config&quot;
content = &quot;${data.template_file.cloudconfig.rendered}&quot;
}
}

data &quot;template_file&quot; &quot;cloudconfig&quot; {
template = &quot;${file(&quot;cloudconfig.tpl&quot;)}&quot;
}
```

And in ``bind/main.tf`` add this (don't forget to change ```` by a secure password) :

```
resource &quot;azurerm_resource_group&quot; &quot;tf-bind&quot; {
name = &quot;RG-bind&quot;
location = &quot;West Europe&quot;

tags = {
Project = &quot;bind&quot;
}
}

resource &quot;azurerm_virtual_network&quot; &quot;tf-vnet&quot; {
name = &quot;vnet-bind&quot;
location = &quot;${azurerm_resource_group.tf-bind.location}&quot;
resource_group_name = &quot;${azurerm_resource_group.tf-bind.name}&quot;
address_space = [&quot;10.0.0.0/16&quot;]

tags = {
Project = &quot;bind&quot;
}
}

resource &quot;azurerm_subnet&quot; &quot;tf-snet&quot; {
name = &quot;main&quot;
resource_group_name = &quot;${azurerm_resource_group.tf-bind.name}&quot;
virtual_network_name = &quot;${azurerm_virtual_network.tf-vnet.name}&quot;
address_prefix = &quot;10.0.0.0/24&quot;
}

resource &quot;azurerm_virtual_machine&quot; &quot;tf-vm-bind&quot; {
count = 1
name = &quot;bind-vm0${count.index}&quot;
location = &quot;${azurerm_resource_group.tf-bind.location}&quot;
resource_group_name = &quot;${azurerm_resource_group.tf-bind.name}&quot;
network_interface_ids = [&quot;${element(azurerm_network_interface.tf-nic.*.id, count.index)}&quot;]
vm_size = &quot;Standard_B1ms&quot;
delete_os_disk_on_termination = true

storage_image_reference {
publisher = &quot;Canonical&quot;
offer = &quot;UbuntuServer&quot;
sku = &quot;16.04-LTS&quot;
version = &quot;latest&quot;
}

storage_os_disk {
name = &quot;dsk-vm0${count.index}&quot;
caching = &quot;ReadWrite&quot;
create_option = &quot;FromImage&quot;
managed_disk_type = &quot;Standard_LRS&quot;
}

os_profile {
computer_name = &quot;bind-vm0${count.index}&quot;
admin_username = &quot;edeneuve&quot;
admin_password = &quot;&quot;
custom_data = &quot;${data.template_cloudinit_config.config.rendered}&quot;
}

os_profile_linux_config {
disable_password_authentication = false
}

tags = {
Project = &quot;bind&quot;
}
}

resource &quot;azurerm_network_interface&quot; &quot;tf-nic&quot; {
count = &quot;1&quot;
name = &quot;nic-vm${count.index}&quot;
resource_group_name = &quot;${azurerm_resource_group.tf-bind.name}&quot;
location = &quot;${azurerm_resource_group.tf-bind.location}&quot;

ip_configuration {
name = &quot;ipconfig&quot;
private_ip_address_allocation = &quot;dynamic&quot;
subnet_id = &quot;${azurerm_subnet.tf-snet.id}&quot;
public_ip_address_id = &quot;${element(azurerm_public_ip.MyResource.*.id, count.index)}&quot;
}

tags = {
Project = &quot;bind&quot;
}
}

resource &quot;azurerm_public_ip&quot; &quot;MyResource&quot; {
count = &quot;1&quot;
name = &quot;pip-vm${count.index}&quot;
resource_group_name = &quot;${azurerm_resource_group.tf-bind.name}&quot;
location = &quot;${azurerm_resource_group.tf-bind.location}&quot;
public_ip_address_allocation = &quot;dynamic&quot;

tags = {
Project = &quot;bind&quot;
}
}
```

In the cloud config template ``bind/files/cloudconfig.tpl``, add this:

```
#cloud-config
package_upgrade: true
packages:
- bind9
- dnsutils
```

Let's deploy !
First connect you to Azure using ``az login``, secondly launch ``terraform init`` from the bind folder, then test your copy/paste with ``terraform plan`` and finally ``terraform apply``. Take a coffee, your Bind server is under construction.

### Bind setup

Connect you on the newly deploy vm using SSH and go to ``/etc/bind/`` and generate a key using ``sudo rndc-confgen``

You should now have a ``rdnc.key`` within the current folder and with some content like :

```
$ sudo cat /etc/bind/rndc.key

key &quot;rndc-key&quot; {
algorithm hmac-md5;
secret &quot;REDACTED&quot;;
};
```

Now, we will edit the config file of Bind9, the main config is ``named.conf``. You need to add a reference to the new ``rndc.key`` :

```
include &quot;/etc/bind/rndc.key&quot;;
```

Then, create new zone in ``named.conf.local`` (You can change the ``toto.int.local.`` by something different :

```
zone &quot;toto.int.local.&quot; {
type master;
file &quot;/etc/bind/zones/db.toto.int.local&quot;;
update-policy {
grant rndc-key zonesub any;
};
};
```

Finally, create a new zone file in a subfolder called ``zones`` (create it) and add the following content in a zone file ``/etc/bind/zones/db.toto.int.local`` (name must match with the "file" directive in the ``named.conf.local``) :

> NS must match your server hostname and records must be correct to.

```
$ORIGIN .
$TTL 3600       ; 1 hour
toto.int.local          IN SOA  ns1.toto.int.local. root.toto.int.local. (
                                2012033116 ; serial
                                3600       ; refresh (1 hour)
                                1800       ; retry (30 minutes)
                                604800     ; expire (1 week)
                                43200      ; minimum (12 hours)
                                )
                        NS      ns1.toto.int.local.
                        NS      ns2.toto.int.local.
$ORIGIN toto.int.local.
$TTL 3600       ; 1 hour
ns1                     A       137.117.141.166
ns2                     A       137.117.141.166
```

As Bind need to create and modify the content in the folder you need to setup right according to :

```
sudo chown -R root:bind ./zones
sudo chmod -R 640 ./zones
```

And then restart bind using :

```
sudo systemctl restart bind9
```

Check the bind logs using :

```
tail /var/log/syslog | grep named

Jun 15 07:44:25 bind-vm00 named[4233]: automatic empty zone: 255.255.255.255.IN-ADDR.ARPA
Jun 15 07:44:25 bind-vm00 named[4233]: automatic empty zone: 0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.IP6.ARPA
Jun 15 07:44:25 bind-vm00 named[4233]: automatic empty zone: 1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.IP6.ARPA
Jun 15 07:44:25 bind-vm00 named[4233]: automatic empty zone: D.F.IP6.ARPA
Jun 15 07:44:25 bind-vm00 named[4233]: automatic empty zone: 8.E.F.IP6.ARPA
Jun 15 07:44:25 bind-vm00 named[4233]: automatic empty zone: 9.E.F.IP6.ARPA
Jun 15 07:44:25 bind-vm00 named[4233]: automatic empty zone: A.E.F.IP6.ARPA
Jun 15 07:44:25 bind-vm00 named[4233]: automatic empty zone: B.E.F.IP6.ARPA
Jun 15 07:44:25 bind-vm00 named[4233]: automatic empty zone: 8.B.D.0.1.0.0.2.IP6.ARPA
Jun 15 07:44:25 bind-vm00 named[4233]: automatic empty zone: EMPTY.AS112.ARPA
Jun 15 07:44:25 bind-vm00 named[4233]: configuring command channel from &#039;/etc/bind/rndc.key&#039;
Jun 15 07:44:25 bind-vm00 named[4233]: command channel listening on 127.0.0.1#953
Jun 15 07:44:25 bind-vm00 named[4233]: configuring command channel from &#039;/etc/bind/rndc.key&#039;
Jun 15 07:44:25 bind-vm00 named[4233]: command channel listening on ::1#953
Jun 15 07:44:25 bind-vm00 named[4233]: managed-keys-zone: loaded serial 13
Jun 15 07:44:25 bind-vm00 named[4233]: zone 0.in-addr.arpa/IN: loaded serial 1
Jun 15 07:44:25 bind-vm00 named[4233]: zone 127.in-addr.arpa/IN: loaded serial 1
Jun 15 07:44:25 bind-vm00 named[4233]: zone 255.in-addr.arpa/IN: loaded serial 1
Jun 15 07:44:25 bind-vm00 named[4233]: zone localhost/IN: loaded serial 2
Jun 15 07:44:25 bind-vm00 named[4233]: zone toto.int.local/IN: loaded serial 2012033116
Jun 15 07:44:25 bind-vm00 named[4233]: all zones loaded
Jun 15 07:44:25 bind-vm00 named[4233]: running
Jun 15 07:44:25 bind-vm00 named[4233]: zone toto.int.local/IN: sending notifies (serial 2012033116)
```

AppArmor need to be updated to let bind change the zones, so you need to set it. To set AppArmor, you need to change the file ``/etc/apparmor.d/usr.sbin.named`` and add the line ``/etc/bind/zones/** rw,`` in this part :

```
cat /etc/apparmor.d/usr.sbin.named
[...]
# /etc/bind should be read-only for bind
# /var/lib/bind is for dynamically updated zone (and journal) files.
# /var/cache/bind is for slave/stub data, since we&#039;re not the origin of it.
# See /usr/share/doc/bind9/README.Debian.gz
/etc/bind/** r,
/etc/bind/zones/** rw,
/var/lib/bind/** rw,
/var/lib/bind/ rw,
/var/cache/bind/** lrw,
/var/cache/bind/ rw,
[...]
```

After the modification restart AppArmor deamon with ``sudo systemctl restart apparmor``, then check the named profile is loaded ``aa-status``.

```
aa-status
apparmor module is loaded.
14 profiles are loaded.
14 profiles are in enforce mode.
/sbin/dient
/usr/bin/lxc-start
/usr/lib/NetworkManager/nm-dhcp-client.action
/usr/lib/NetworkManager/nm-dhcp-helper
/usr/lib/connman/scripts/dient-script
/usr/lib/lxd/lxd-bridge-proxy
/usr/lib/snapd/snap-confine
/usr/lib/snapd/snap-confine//mount-namespace-capture-helper
/usr/sbin/named
/usr/sbin/tcpdump
lxc-container-default
lxc-container-default-cgns
lxc-container-default-with-mounting
lxc-container-default-with-nesting
0 profiles are in complain mode.
2 processes have profiles defined.
0 processes are in enforce mode.
0 processes are in complain mode.
2 processes are unconfined but have a profile defined.
/sbin/dient (921)
/usr/sbin/named (6523)
```

## Dynamic DNS Update With Terraform

Now we will work in the ``dns``folder :

```
- dns
- main.tf
- provider.tf
```

In ``provider.tf``, add this :

```
provider &quot;dns&quot; {
update {
server = &quot;&quot;
key_name = &quot;&quot;
key_algorithm = &quot;hmac-md5&quot;
key_secret = &quot;&quot;
}
}
```

Then from the dns folder launch ``terraform init``, to grab the dns plugin.

In the ``main.tf`` file add Terraform resources :

```
resource &quot;dns_a_record_set&quot; &quot;www&quot; {
zone = &quot;toto.int.local.&quot;
name = &quot;www&quot;

addresses = [
&quot;192.168.0.1&quot;,
&quot;192.168.0.2&quot;,
&quot;192.168.0.3&quot;,
]

ttl = 300
}

resource &quot;dns_cname_record&quot; &quot;foo&quot; {
zone = &quot;toto.int.local.&quot;
name = &quot;foo&quot;
cname = &quot;tata.toto.int.local.&quot;
ttl = 300
}

resource &quot;dns_a_record_set&quot; &quot;xxx&quot; {
zone = &quot;toto.int.local.&quot;
name = &quot;tata&quot;
addresses = [&quot;192.168.0.1&quot;]
ttl = 300
}
```

Then plan with ``terraform plan`` :

```
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
+ create

Terraform will perform the following actions:

+ dns_a_record_set.www
id:
addresses.#: &quot;3&quot;
addresses.1737095236: &quot;192.168.0.3&quot;
addresses.2307365224: &quot;192.168.0.1&quot;
addresses.277792978: &quot;192.168.0.2&quot;
name: &quot;www&quot;
ttl: &quot;300&quot;
zone: &quot;toto.int.local.&quot;

+ dns_a_record_set.xxx
id:
addresses.#: &quot;1&quot;
addresses.2307365224: &quot;192.168.0.1&quot;
name: &quot;tata&quot;
ttl: &quot;300&quot;
zone: &quot;toto.int.local.&quot;

+ dns_cname_record.foo
id:
cname: &quot;tata.toto.int.local.&quot;
name: &quot;foo&quot;
ttl: &quot;300&quot;
zone: &quot;toto.int.local.&quot;

Plan: 3 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn&#039;t specify an &quot;-out&quot; parameter to save this plan, so Terraform
can&#039;t guarantee that exactly these actions will be performed if
&quot;terraform apply&quot; is subsequently run.
```

You can now apply with ``terraform apply``. Check the dns log with ``cat /var/log/syslog | grep named`` :

```
Jun 15 09:02:24 bind-vm00 named[6523]: client XX.xx.XX.XX#61068/key rndc: updating zone &#039;toto.int.local/IN&#039;: deleting rrset at &#039;www.toto.int.local&#039; A
Jun 15 09:02:24 bind-vm00 named[6523]: zone toto.int.local/IN: sending notifies (serial 2012033123)
Jun 15 09:02:24 bind-vm00 named[6523]: client XX.xx.XX.XX#61066/key rndc: updating zone &#039;toto.int.local/IN&#039;: deleting rrset at &#039;foo.toto.int.local&#039; CNAME
Jun 15 09:02:24 bind-vm00 named[6523]: client XX.xx.XX.XX#61067/key rndc: updating zone &#039;toto.int.local/IN&#039;: deleting rrset at &#039;tata.toto.int.local&#039; A
Jun 15 09:02:29 bind-vm00 named[6523]: zone toto.int.local/IN: sending notifies (serial 2012033125)
```

## Test your new record

To validate that the dns have created the new records, you can use your workstation and the ``nslookup`` command (from Windows, Linux or macOs)

```
nslookup -

&gt; www.toto.int.local
&gt; www.toto.int.local.
Serveur : UnKnown
Address: 51.144.181.124

Nom : www.toto.int.local
Addresses: 192.168.0.2
192.168.0.1
192.168.0.3
```

Now, you can create a DNS Module and create your VM and add them in your new nice Bind9 server.