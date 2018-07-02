---
ID: 106
post_title: >
  Installation de Nginx en reverse proxy
  avec réecriture du HTML
author: etienne.deneuve
post_excerpt: ""
layout: post
permalink: >
  https://etienne.deneuve.xyz/2017/02/02/installation-de-nginx-en-reverse-proxy-avec-reecriture-du-html/
published: true
post_date: 2017-02-02 22:30:06
---
<h3>Nginx</h3>
Nginx est un serveur Web modulaire qui fonctionne très bien en reverse proxy et possède de nombreuses possibilités. De plus il est très léger, je vous invite à consulter leur site : <a href="http://nginx.org/">www.nginx.org</a>
<h3>La problématique</h3>
Un client avait un site internet assez mal écrit avec beaucoup d'url en dur dans le code et qui aurait nécessité beaucoup trop de temps pour le remettre en conformité.
l'idée est de ré-écrire les urls dans les pages transmises par le serveur Web, en l'occurrence un serveur Apache (pas à jour !).
Nous aurions pu utiliser un vrai reverse proxy (Kemp, F5..) afin de faire la même chose mais le client n'en dispose pas et l'investissement pour une seule machine virtuelle était un peu surdimensionné. Si la  machine était dans <a href="https://azure.microsoft.com/fr-fr/">Azure</a>, nous aurions pu utiliser les nombreuses briques présentes dans la solution de Cloud Microsoft. l'idée est donc de ré-écrire les pages à la volées lors de la transmission avec le client.
<h3>Solution mise en place</h3>
Nous avons choisi d'installer un serveur Nginx sur une machine virtuelle Centos 7, cette installation est donc reproductible chez vous sur un système Centos, Red Hat ou Fedora
<h4>Installation de Nginx</h4>
Nginx est présent dans les dépots Centos mais la version proposée ne dispose pas du module de substitution dont nous avons besoin. Il faut donc passer par la compilation, et donc installer les outils nécessaires comme ci-dessous :
<pre>yum install gcc-c++ \
                  pcre-devel \
                  zlib-devel \
                  make \
                  wget \
                  openssl-devel \
                  libxml2-devel \
                  libxslt-devel \
                  gd-devel \
                  perl-ExtUtils-Embed \
                  GeoIP-devel \
                  gperftools-devel \
                  git
</pre>
Ensuite on récupère les sources de Nginx depuis le site de Nginx :
<pre>cd /tmp
wget http://nginx.org/download/nginx-1.10.1.tar.gz
tar -xvf nginx-1.10.1.tar.gz
</pre>
Comme l'extension "http subsitution filter" module n'est pas un module "officiel", il faut le récupérer sur le GitHub de yaoweibin :
<pre>git clone git://github.com/yaoweibin/ngx_http_substitutions_filter_module.git  
</pre>
Ensuite on configure le make :
<pre>  cd nginx-1.10.1
  ./configure \
        --user=nginx                          \
        --group=nginx                         \
        --prefix=/etc/nginx                   \
        --sbin-path=/usr/sbin/nginx           \
        --conf-path=/etc/nginx/nginx.conf     \
        --pid-path=/var/run/nginx.pid         \
        --lock-path=/var/run/nginx.lock       \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --with-http_gzip_static_module        \
        --with-http_stub_status_module        \
        --with-http_ssl_module                \
        --with-pcre                           \
        --with-file-aio                       \
        --with-http_realip_module             \
        --without-http_scgi_module            \
        --without-http_uwsgi_module           \
        --without-http_fastcgi_module \
        --add-module=/tmp/ngx_http_substitutions_filter_module      
</pre>
On lance le make
<pre>make
make install
</pre>
<h4>Configuration de Nginx</h4>
Il faut créer un compte dédié a Nginx, faire tourner un serveur Web en root étant largement déconseillé (oui oui, il parait) :
<pre>useradd -r nginx</pre>
Comme notre Nginx est une version compilée, nous devons créer un fichier de service conforme à systemd :
<pre>echo "[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target" &gt; /lib/systemd/system/nginx.service
</pre>
Comme on est pas des gros bourrins, on configure firewalld via firewall-cmd :
<pre>firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload
</pre>
Et bien sur on vérifie que les règles sont appliquées :
<pre>iptables-save | grep 80</pre>
<h4>Création de la règles de réécriture :</h4>
Par défaut, le fichier de configuration Nginx est situé dans le dossier que nous avons mentionné dans la partie "--prefix=/etc/nginx",  nginx.conf
Remplacez la partie "server {...}" par celle ci :
<pre>    server {
        listen       80;
        server_name     localhost
                        old.domainquonveut.com;

        access_log  /var/log/nginx/oldsite.access.log  main;
        error_log /var/log/nginx/oldsite.error.log  main;
        location / {
                proxy_pass      http://ancienserveur:80; #ipduserveur
                proxy_set_header Host www.vhostduapache.com;
                proxy_set_header Accept-Encoding "";
                subs_filter www.domaineendurdanslecodequinousplaitpas.com old.domainquonveut.com gi;
                subs_filter www.domaineendurdanslecodequinousplaitpas.fr old.domainquonveut.fr gi;
        }
    }
</pre>
<ol>
 	<li>proxy_pass : c'est l'option qui permet de forwarder le traffic entrant vers le serveur Apache</li>
 	<li>proxy_set_header : c'est l'option qui permet de modifier le header envoyé au serveur Web
<ul>
 	<li>Host : dans notre cas on veut changer le Host présent dans le header afin de corrigé celui appelé par l'utilisateur.</li>
 	<li>Accept-Encoding : On utilise également "Accept-Encoding "";" afin de forcer le serveur Apache a ne pas utiliser la compression gzip des pages, car sinon, le module ne pourra pas ré-récrire les objets.</li>
</ul>
</li>
 	<li>subs_filter : la syntaxe est très simple "stringquidoitdisparaitre" "stringquidoitapparaitre" gi
le paramètre gi sert à indiquer que nous souhaitons ré-écrire tout (toute les occurrences), le i pour case insensitive, afin de le récupérer sans tenir compte de la casse.</li>
</ol>
Dans la partie "Accept-Encoding", je mentionne qu'il faut désactiver la compression gzip, pour deux raisons :
<ol>
 	<li>Niveau ressources, ca sert à rien de compresser deux fois la même donnée, ca sera pas plus petit (bah ouais...)</li>
 	<li>Le module de substitution ne gère pas la décompression à la volée, puisqu'il analyse les fichiers plats et remplace les occurences des strings qu'on souhaite dégager.</li>
 	<li>Lorsqu'on mets en place un reverse proxy, on peut vouloir mettre en cache certains éléments pour décharger les serveurs web, afin de pas leurs demander 10 fois à la seconde la même image (le logo de la boite par exemple), dès lors si les serveurs web envoient du flux compressé, il ne pourra pas le faire correctement.</li>
</ol>
<h3>Les cas d'usage</h3>
Utiliser ce type de module peut être interressant, par exemple, sur un Wordpress de test. En effet, Wordpress stocke dans la base l'url d'accès, et l'envoie dans quelques pages, rien de bien méchant! Du coup, avec cette petite manipulation, vous pouvez avoir un Wordpress de test qui serait une copie 100% identique à votre prod', juste avec un petit copier coller de la VM ou une autre technique :)

Dans le cas de mon blog, si je souhaite mettre en place une copie de mon site avec par exemple comme url "test.deneuve.xyz" sans rien toucher (en dehors d'une petite copie) je devrai faire un nginx.conf tel que :
<pre>server {
        listen       80;
        server_name     localhost
                        test.deneuve.xyz;

        access_log  /var/log/nginx/test.access.log  main;
        error_log /var/log/nginx/test.error.log  main;
        location / {
                proxy_pass      http://192.168.1.150:80; #ipduserveurbidon
                proxy_set_header Host etienne.deneuve.xyz;
                proxy_set_header Accept-Encoding "";
                subs_filter etienne.deneuve.xyz test.deneuve.xyz gi;
                
        }
    }</pre>
&nbsp;