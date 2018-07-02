---
ID: 73
post_title: 'Utiliser Ansible et Azure, oui c&#8217;est possible !'
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2017/01/26/utiliser-ansible-et-azure-oui-cest-possible/
published: true
post_date: 2017-01-26 18:54:27
---
<h2>Avant Propos</h2>
<a href="https://www.ansible.com/"><u>Ansible</u></a> est un outil d’orchestration et déploiment proposé par <a href="https://www.redhat.com/fr"><u>Red Hat</u></a>. Il est utilisé par de nombreuses entreprises dans le domaine de l’Open Source. Pendant la préparation des démos pour le <a href="https://mscloudsummit.fr/fr/accueil/"><u>MS Cloud Summit</u></a>, <a href="https://stanislas.io/"><u>Stanislas Quastana</u></a> et moi-même avons donc décidé de présenter les outils Open Source Terraform et Ansible afin de déployer des services dans Azure. Vous pouvez retrouver tous les détails de notre session “Infrastructures dans Azure : C’est comme du lego” : <a href="http://aka.ms/cloudsummitlego"><u>ici</u></a>
<h2>Ansible</h2>
Ansible est donc un outil de gestion des configurations, je ne vais pas vous refaire un tutorial d’installation vu le nombre déjà disponible avec votre ami <a href="https://www.bing.com/search?q=tutorial+installation+ansible&amp;go=Envoyer&amp;qs=n&amp;form=QBLH&amp;sp=-1&amp;pq=tutorial+installation+ansible&amp;sc=3-24&amp;sk=&amp;cvid=12AEAA43E790440088D942223E35173A"><u>Bing</u></a> ou <a href="https://www.google.fr/#q=tutorial+install+ansible"><u>Google</u></a>.

En revanche l’installation des éléments nécessaires pour Azure peuvent être un peu complexe à installer et les documentations peu présentes. J’ai décidé d’installer Ansible sur une machine dans Azure que j’ai déployé avec Terraform, vous pouvez trouver un fichier d’exemple dans mon repo Azure sur <a href="https://github.com/EtienneDeneuve/Azure/tree/master/Terraform/01%20-%20IaaS"><u>GitHub</u></a> afin de créer une machine virtuelle prête à l’emploi. Mon exemple déploie une machine Ubuntu 16.04-LTS et mon installation fonctionnera sur d’autres machines du même type.
<h3>Installation de dépendances</h3>
Première chose à faire sur une nouvelle machine c’est bien entendu les mises à jour :
<pre>sudo apt-get update</pre>
Ensuite il faut installer les dépendances nécessaires a Ansible à savoir Phython :
<pre>sudo apt-get install software-properties-common</pre>
Il faut également installer des librairies de convention et ssl :
<pre>sudo apt-get install libffi-dev libssl-dev</pre>
<h3>Installation de Ansible</h3>
Sur Ubuntu le repository PPA propose une version de Ansible il faut donc simplement l’ajouter et faire une petite mise à jour du cache apt-get:
<pre>sudo apt-add-repository ppa:ansible/ansible
 sudo apt-get update</pre>
Lorsqu’on utilise Ansible ou d’autres outils de configuration management il est intéressant d’utiliser git ainsi que sshpass on installera donc les trois paquets :
<pre>sudo apt-get install ansible sshpass git</pre>
<h3>Installation des prérequis pour Azure</h3>
Pour installer les éléments nécessaires, pip est assez pratique :
<pre>sudo apt-get install python-pip
 sudo pip install --upgrade pip 
</pre>
Le paquet urllib3 fourni dans les repository d’Ubuntu n’est pas installé de la bonne manière pour les dépendances Azure et donc il faut le mettre à jour par pip :
<pre>sudo pip install urllib3 --upgrade</pre>
Ensuite on installe MS rest …
<pre>sudo pip install msrest==0.4.4</pre>
et MS Rest Azure …
<pre>sudo pip install msrestazure==0.4.4</pre>
Puis le SDK Python pour Azure :
<pre>sudo pip install azure==2.0.0rc5</pre>
<h3>Configuration de Ansible (a minima)</h3>
Première chose à faire comme toujours lorsqu’on modifie des fichiers sur un serveur c’est le backup du fichier de configuration :
<pre>mv /etc/ansible/ansible.cfg /etc/ansible/ansible.cfg.backup
</pre>
J’ai glané sur le web quelques modifications à faire dans le fichier de configuration et on ajoute la machine locale dans le fichier hosts de Ansible :
<pre>printf "[defaults]\nhost_key_checking = False\n\n" \ 
        &gt;&gt; /etc/ansible/ansible.cfg 
 echo '[ssh_connection]\ncontrol_path = ~/.ssh/ansible-%%h-%%r' \
        &gt;&gt; /etc/ansible/ansible.cfg
 echo "\nscp_if_ssh=True" &gt;&gt; /etc/ansible/ansible.cfg
 echo "localhost" &gt; etc/ansible/hosts</pre>
<h3>Azure !</h3>
Pour tester que notre configuration fonctionne correctement on va utiliser un module disponible sur le GitHub de Ansible, azure_rm.py et son fichier de configuration azure_rm.ini. Mais avant, préparons quelques dossiers pour ranger les Playbook Ansible et les scripts d’inventaire.
<pre>mkdir -p /opt/azure/playbook 
mkdir -p /opt/azure/inventory
mkdir -p ~/azure</pre>
Il faut télécharger le module :
<pre>cd /opt/azure/inventory
wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/azure_rm.py
wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/azure_rm.ini
chmod +x /opt/azure/inventory/azure_rm.py</pre>
Ansible a besoin tout comme Terraform d’un Service Principal sur votre tenant Azure afin de d’accéder à vos ressources, je ne vais détailler la méthode ici, rendez-vous sur le blog de <a href="https://stanislas.io/2017/01/02/modeliser-deployer-et-gerer-des-ressources-azure-avec-terraform-de-hashicorp/"><u>Stanislas</u></a> afin d’avoir la méthode qui est exactement la même. Une fois ces informations collectées, créer un fichier « .credentials » dans votre $Home/azure :
<pre>cd ~ /azure/
touch .credentials</pre>
et renseigner le contenu tel que ci dessous :
<pre>[default]
subscription_id=YOUR-SUBID
 client_id=YOUR-ClientID
 secret=SECRET
 tenant=TENANTID</pre>
Attention : vous devez indiquer les renseignements sans placer de quote au quels cas Azure vous refusera l’authentification. Pensez également à ne jamais diffuser vos clés, elles permettent d’avoir accès à votre tenant de manière similaire a un administrateur !
<h3>Le test !</h3>
Pour vérifier que tout est bien configuré, nous allons créer un playbook basé sur les informations de l’inventaire que nous avons téléchargé. Avant de faire votre premier playbook il est peut-être nécessaire de faire un petit rappel sur les fichier YML ou YAML.

Les fichiers YML (Yet Another Language Markup) est un langage simple mais qui comporte quelques subtilités qui peuvent se révéler frustrantes au départ. En effet les tabulations doivent être de 2 espaces. J’utilise <a href="https://code.visualstudio.com/"><u>Visual Studio Code</u></a> qui comporte la possibilité d’être configuré afin de ne pas être ennuyé, de plus si vous travaillez sur un Pc sous Windows, les fichiers doivent être en retour de ligne LF et non CRLF! Je détaillerai très prochainement ma configuration et mes plugins que j’utilise sur VS Code.
Le playbook :
<pre> - name: Test the inventory script
  hosts: azure
  connection: local
  gather_facts: no
  tasks:
    - debug: msg="{{ inventory_hostname }} has powerstate {{ powerstate }}"

</pre>
On l’enregistre dans /opt/azure/playbook/test.yml puis on le lance avec la commande :

[code language=bash]

ansible-playbook -i <span class="pl-smi">/opt/azure/inventory/azure_rm.py <span class="pl-smi">/opt/azure/playbook/test.yml</span></span>

[/code]

Vous obtiendrez normalement un retour de ce type :
<pre>PLAY [Test the inventory script] ***********************************************
 
TASK [debug] *******************************************************************
ok: [ansiblevm01] =&gt; {     
    "msg": "ansiblevm01 has powerstate running"
}
ok: [RG-Terraform-Ansible-vm] =&gt; {      
    "msg": "RG-Terraform-Ansible-vm has powerstate running"
}
ok: [Blog] =&gt; {       
    "msg": "Blog has powerstate running"
}
 
PLAY RECAP *********************************************************************
Blog                    : ok=1 changed=0 unreachable=0 failed=0
RG-Terraform-Ansible-vm : ok=1 changed=0 unreachable=0 failed=0
ansiblevm01             : ok=1 changed=0 unreachable=0 failed=0</pre>
Vous devez retrouver vos ressources groupes déjà présents dans votre tenant Azure !!

Tout est donc prêt pour pouvoir déployer des ressources dans Azure et les configurer.