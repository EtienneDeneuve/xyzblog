---
ID: 419
title: >
  Open Source + Windows = Vs Code + Bash +
  Git
author: etienne.deneuve
post_excerpt: ""
layout: layouts/post-sidebar.njk
mySlug: setup-vs-code-bash-git
permalink: "{{ page.date | date: '%Y/%m/%d' }}/{{ mySlug }}/index.html"

published: true
date: 2018-06-26 13:03:40
---
Dans cet article, je vous expose mon setup pour Terraform et Ansible, avec git depuis un poste Windows 10.

<!--more-->Si vous ne connaissez pas encore Git, je vous invite à lire mes articles à ce sujet :
- [Git for ops](https://etienne.deneuve.xyz/2018/06/23/git-pour-ops-par-un-ops/)
- [Git for ops, le retour](https://etienne.deneuve.xyz/2018/06/28/git-pour-les-ops-par-un-ops-le-retour/)
Si c'est Visual Studio Code c'est par [ici](https://etienne.deneuve.xyz/2017/01/26/visual-studio-code-pour-ansible-terraform/)

Et sinon, pour aller plus loin, lisez mon article sur [Documentation as Code](https://etienne.deneuve.xyz/2018/06/26/documentation-as-code/)


## Visual Studio Code

Créer un nouveau workspace :

Ouvrez un dossier git ou un vide puis sauvegarder le workspace via "File --&gt; Save workspace as"

Dans la configuration du workspace Crtl + P : " &gt; Open Workspace Settings"  et ajoutez :

```json
settings: {
&quot;terminal.integrated.shell.windows&quot;: &quot;c:\\windows\\System32\\bash.exe&quot;
}
```

Et relancez VS Code ;)
<h2>Configuration de Bash</h2>
Il existe pleins de tuto pour installer Bash sur Windows, je ne reviens pas là-dessus, mais voici tout de même un liens vers le store pour un <a href="https://www.microsoft.com/store/productId/9NBLGGH4MSV6">Ubuntu </a>

Je vous donne un petit script bash pour installer les différents packages que j'utilise pour Bash, vous pouvez également installer <a href="https://github.com/Bash-it/bash-it">bash-it</a> :

```
#!/bin/bash
echo &quot;installation des packages de puis les depots ubuntu&quot;
sudo apt install git wget unzip python curl python-dev build-essential -q -y
echo &quot;Téléchargement de pip&quot;
wget https://bootstrap.pypa.io/get-pip.py
echo &quot;Téléchargement de Terraform&quot;
wget https://releases.hashicorp.com/terraform/0.11.7/terraform_0.11.7_linux_amd64.zip -O terraform.zip
echo &quot;Téléchargement de Terraform Docs&quot;
wget https://github.com/segmentio/terraform-docs/releases/download/v0.3.0/terraform-docs_linux_amd64 -O terraform-docs
echo &quot;Ajout de Terraform et Terraform-docs dans /home/${USER}/.local/bin/&quot;
unzip terraform.zip
mv terraform /home/${USER}/.local/bin/
mv terraform-docs /home/${USER}/.local/bin/
echo &quot;installation de pip en mode user&quot;
python get-pip.py --user
echo &quot;verification du path&quot;
if [[ &quot;:$PATH:&quot; == *&quot;:$HOME/.local/bin&quot;* ]]; then
echo &quot;Your path is correctly set&quot;
else
echo &quot;Ajout de /home/${USER}/.local/bin dans le path&quot;
PATH=$PATH:/home/${USER}/.local/bin
export PATH
fi
echo &quot;Mise a jour de pip par pip&quot;
pip install pip --upgrade --user
echo &quot;installation des outils pour ansible&quot;
pip install ansible-lint ansible-docgen pre-commit ansible[azure] pywinrm molecule --user
```

## Configuration de Pre-Commit

Pre-commit permet d'exécuter des petits tests sur le code lors du commit dans un hooks "pre-commit".

Dans le repo Git, ajoutez un fichier .pre-commit-config.yaml, c'est lui qui indiquera à pre-commit ce qu'il doit configurer dans git (.git/hooks) :

Wordpress ne conservant pas bien les espaces je vous invite a récupérer une version sur mon GitHub <a href="https://raw.githubusercontent.com/EtienneDeneuve/vsts-for-ops/master/.pre-commit-config.yaml">ici</a>

```
cat &lt;./.pre-commit-config.yaml
---
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
rev: v1.3.0
hooks:
- id: check-yaml
- id: check-xml
- id: check-json
- id: end-of-file-fixer
- id: check-case-conflict
- id: check-merge-conflict
#- id: check-executables-have-shebangs
- id: check-added-large-files
- id: trailing-whitespace
- id: detect-private-key
- id: pretty-format-json
- id: sort-simple-yaml
- repo: https://github.com/willthames/ansible-lint.git
rev: v3.5.0rc1
hooks:
- id: ansible-lint
files: \.(yaml|yml)$
exclude: ./env/
- repo: git://github.com/antonbabenko/pre-commit-terraform
rev: v1.7.3
hooks:
- id: terraform_fmt
- id: terraform_docs
EOF
```

Puis on l'ajoute à l'index :

```
git add .pre-commit-config.yaml
```

On commit :

```
git commit . -m &quot;validation des pré commit hooks&quot; -n
git push origin master
```

En suite, on valide l'installation des éléments voulu avec :

```
pre-commit install
pre-commit installed at /mnt/c/Users/etien/Documents/it/blog/.git/hooks/pre-commit
```

et on run sur les fichiers existant :

```
2018-06-23 10:03:04 ⌚ etienne in /mnt/c/Users/etien/Documents/it/blog
± | master S:7 U:1 ✗| → pre-commit run --all-files
Check Yaml...............................................................Passed
Check Xml............................................(no files to check)Skipped
Check JSON...........................................(no files to check)Skipped
Fix End of Files.........................................................Passed
Check for case conflicts.................................................Passed
Check for merge conflicts................................................Passed
Check for added large files..............................................Passed
Trim Trailing Whitespace.................................................Passed
Detect Private Key.......................................................Passed
Pretty format JSON...................................(no files to check)Skipped
Sort simple YAML files...............................(no files to check)Skipped
Ansible-lint.............................................................Passed
Terraform fmt............................................................Passed
Terraform docs...........................................................Passed
```

## Test avec git

Ajoutez un fichier à l'index avec ``git add fichier.yml`` avec ce contenu :

```
---
- name: Check Installed Updates
win_updates:
category_names:
- SecurityUpdates
- CriticalUpdates
- UpdateRollups
state: searched
log_path: c:\ansible_wu1.txt
register: search_updates
tags:
- updates
```

J'ai volontairement ajouté des espaces qui ne sont peut-être pas visible avec Wordpress, dans ce cas utiliser un playbook ansible <a href="https://raw.githubusercontent.com/EtienneDeneuve/Azure/master/Ansible/inventory/init.yml">ici</a>

Et tentez de faire un commit :

```
git commit .
Check Yaml...............................................................Passed
Check Xml............................................(no files to check)Skipped
Check JSON...........................................(no files to check)Skipped
Fix End of Files.........................................................Passed
Check for case conflicts.................................................Passed
Check for merge conflicts................................................Passed
Check for added large files..............................................Passed
Trim Trailing Whitespace.................................................Failed
hookid: trailing-whitespace

Files were modified by this hook. Additional output:

Fixing test.yml

Detect Private Key.......................................................Passed
Pretty format JSON...................................(no files to check)Skipped
Sort simple YAML files...............................(no files to check)Skipped
Ansible-lint.............................................................Passed
Terraform fmt........................................(no files to check)Skipped
Terraform docs.......................................(no files to check)Skipped
```

Le hook de pre-commit trim trailing whitespace a nettoyer les espaces en trop dans votre fichier. Il vous faudra donc ajouter à nouveau le fichier de test à l'index avec un ``git add test.yml``, puis faite un nouveau commit :

```
git add test.yml
git commit . -m &quot;Test&quot;
Check Yaml...............................................................Passed
Check Xml............................................(no files to check)Skipped
Check JSON...........................................(no files to check)Skipped
Fix End of Files.........................................................Passed
Check for case conflicts.................................................Passed
Check for merge conflicts................................................Passed
Check for added large files..............................................Passed
Trim Trailing Whitespace.................................................Passed
Detect Private Key.......................................................Passed
Pretty format JSON...................................(no files to check)Skipped
Sort simple YAML files...............................(no files to check)Skipped
Ansible-lint.............................................................Passed
Terraform fmt........................................(no files to check)Skipped
Terraform docs.......................................(no files to check)Skipped
[master 8a6cf31] Adding test.yml
1 files changed, 10 insertions(+), 0 deletions(-)
create mode 100644 test.yml
```

Voila, maintenant vous savez comment installer des hooks de pre-commit pour faire une pré-validation.

Pour aller un peu plus loin, je vous invite à lire mon article sur la gestion automatique de la documentation avec Terraform-Docs et Ansible-Docgen.