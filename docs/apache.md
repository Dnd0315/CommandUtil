# Apache (serveur web)

[← Index](../README.md)

Apache 2.4. Debian / Ubuntu : paquet et service `apache2`. RHEL : paquet et service `httpd`. Équivalent pour nginx : voir [Nginx](nginx.md).

## Installation et fichiers

- `sudo apt install apache2` → installe et démarre le serveur (RHEL : `dnf install httpd`, puis activer le service, voir [systemd](systemd.md)) ; test : la page « Apache2 Default Page » doit répondre en 200 (`curl -I`, voir [Nginx](nginx.md))
- Debian / Ubuntu :
  - `/etc/apache2/apache2.conf` → configuration principale ; `/etc/apache2/ports.conf` → ports d'écoute (`Listen 80`)
  - `/etc/apache2/sites-available/` (sites disponibles) et `sites-enabled/` (liens vers les sites ACTIFS)
  - `/etc/apache2/mods-available/` et `mods-enabled/` (modules) ; `conf-available/` et `conf-enabled/` (fragments de configuration)
  - `/etc/apache2/envvars` → variables comme `APACHE_LOG_DIR` ; utilisateur du serveur : `www-data`
- RHEL : `/etc/httpd/conf/httpd.conf`, fragments dans `/etc/httpd/conf.d/*.conf`, logs dans `/var/log/httpd/`, utilisateur `apache`
- `/var/www/html` → racine du site par défaut (DocumentRoot) ; `/var/log/apache2/access.log` et `error.log` (RHEL : `/var/log/httpd/`)
- Les fichiers servis doivent être lisibles par l'utilisateur du serveur (`chown -R www-data:www-data /var/www/example`, voir [Permissions](permissions.md))

## Commandes de gestion (Debian / Ubuntu)

- `sudo apache2ctl configtest` (ou `apache2ctl -t`) → vérifie la syntaxe : « Syntax OK ». À faire AVANT chaque rechargement (RHEL : `httpd -t`)
- `sudo apache2ctl -S` → affiche les hôtes virtuels reconnus, leur fichier et le site par défaut (RHEL : `httpd -S`) : le premier réflexe quand le mauvais site répond
- `apache2ctl -M` → modules chargés (RHEL : `httpd -M`)
- `sudo a2ensite example.conf` / `sudo a2dissite example.conf` → active / désactive un site (crée / supprime le lien dans sites-enabled)
- `sudo a2enmod rewrite ssl headers` / `sudo a2dismod rewrite` → active / désactive des modules
- `sudo a2enconf servername` / `sudo a2disconf servername` → idem pour conf-available
- Appliquer : `reload` du service (voir [systemd](systemd.md)) ; `sudo apache2ctl graceful` → redémarrage en douceur qui laisse finir les requêtes en cours
- « AH00558 : Could not reliably determine the server's fully qualified domain name » → avertissement sans gravité : créer `/etc/apache2/conf-available/servername.conf` contenant `ServerName localhost` puis `a2enconf servername`

## Hôtes virtuels (plusieurs sites sur un serveur)

Créer `/etc/apache2/sites-available/example.conf` :

```apache
<VirtualHost *:80>
    ServerName www.example.lan
    ServerAlias example.lan
    ServerAdmin admin@example.lan
    DocumentRoot /var/www/example

    ErrorLog ${APACHE_LOG_DIR}/example-error.log
    CustomLog ${APACHE_LOG_DIR}/example-access.log combined

    <Directory /var/www/example>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```

Étapes : 1) `sudo mkdir -p /var/www/example` + un `index.html` ; 2) créer le fichier ci-dessus ; 3) `a2ensite example.conf` ; 4) `apache2ctl configtest` ; 5) `reload` ; 6) tester :

- `curl -H "Host: www.example.lan" http://localhost` → simule la requête d'un client qui demande ce nom (sinon ajouter le nom à /etc/hosts, voir [Réseau](reseau.md))
- Hôtes virtuels par NOM : plusieurs `<VirtualHost *:80>` avec des `ServerName` différents ; le PREMIER chargé répond quand aucun nom ne correspond (visible avec `apache2ctl -S`)
- Hôtes virtuels par PORT : ajouter `Listen 8080` dans ports.conf, puis `<VirtualHost *:8080>`. Sous SELinux, autoriser aussi le port (`semanage port`, voir [SELinux / AppArmor](selinux-apparmor.md)) et le pare-feu
- Pare-feu Ubuntu : `sudo ufw app list` → profils d'applications connus ; `sudo ufw allow 'Apache Full'` → ouvre 80 ET 443 (`'Apache'` : 80 seulement ; `'Apache Secure'` : 443 seulement). RHEL : voir [Équivalents RHEL](rhel-fedora.md)

## Directives utiles

- `DocumentRoot` racine du site / `DirectoryIndex index.php index.html` fichiers d'index essayés dans l'ordre / `ServerName` nom principal / `ServerAlias` autres noms
- `Options -Indexes` interdit la liste du contenu d'un dossier sans index (`+Indexes` l'autorise) / `FollowSymLinks` suit les liens symboliques
- `Alias /docs /usr/share/doc` → publie un dossier situé ailleurs sous /docs (il lui faut aussi un bloc `<Directory>` avec `Require all granted`)
- `ErrorDocument 404 /404.html` → page d'erreur personnalisée / `Redirect permanent /ancien /nouveau` → redirection
- `<Directory chemin>` (un dossier du disque) / `<Location /url>` (une URL) / `<Files "*.bak">` (des fichiers) : portées des règles d'accès
- `AllowOverride None` ignore les fichiers `.htaccess` (plus rapide) ; `AllowOverride All` les autorise (ou `AuthConfig` seulement pour l'authentification)

## Restreindre l'accès à une page

Syntaxe Apache 2.4 : `Require` (les anciens `Order` / `Allow` / `Deny` de la 2.2 ne s'utilisent plus).

```apache
# par adresse IP
<Directory /var/www/example/admin>
    Require ip 10.0.0.0/24
</Directory>

# par mot de passe
<Directory /var/www/example/prive>
    AuthType Basic
    AuthName "Zone privée"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
</Directory>

# IP OU mot de passe / IP ET mot de passe
<Directory /var/www/example/mixte>
    AuthType Basic
    AuthName "Zone mixte"
    AuthUserFile /etc/apache2/.htpasswd
    <RequireAny>
        Require ip 10.0.0.0/24
        Require valid-user
    </RequireAny>
</Directory>
```

- `Require all granted` / `Require all denied` → tout autoriser / tout refuser ; `Require not ip 10.0.0.5` (dans un `<RequireAll>`) → exclure une adresse ; `Require user john` → un utilisateur précis ; `<RequireAll>` = toutes les conditions ET, `<RequireAny>` = au moins une (par défaut quand on en met plusieurs)
- `sudo apt install apache2-utils` → fournit `htpasswd` (RHEL : paquet `httpd-tools`)
- `sudo htpasswd -c /etc/apache2/.htpasswd john` → crée le fichier de mots de passe avec john (⚠️ `-c` = créer : sur un fichier existant, il est ÉCRASÉ) ; sans `-c` pour ajouter : `sudo htpasswd /etc/apache2/.htpasswd jane`
- `sudo htpasswd -D /etc/apache2/.htpasswd jane` → supprime un utilisateur ; `htpasswd -b fichier user motdepasse` → mot de passe donné sur la ligne de commande (visible dans l'historique : à éviter)
- Fichier .htaccess (règles placées DANS le dossier, sans toucher à la config du serveur) : mêmes directives, mais `AllowOverride AuthConfig` (ou `All`) est obligatoire dans le `<Directory>` correspondant
- Tester : `curl -I http://localhost/prive/` → `401 Unauthorized` ; `curl -u john:motdepasse http://localhost/prive/` → 200 ; un refus par IP donne `403 Forbidden`

## HTTPS (mod_ssl)

Certificat et clé : voir [SSL / TLS](ssl-tls.md) (la clé privée doit être lisible par root seulement, ex : `/etc/ssl/private/example.key`).

```apache
<VirtualHost *:443>
    ServerName www.example.lan
    DocumentRoot /var/www/example
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/example.crt
    SSLCertificateKeyFile /etc/ssl/private/example.key
</VirtualHost>

<VirtualHost *:80>
    ServerName www.example.lan
    Redirect permanent / https://www.example.lan/
</VirtualHost>
```

- `sudo a2enmod ssl` puis `a2ensite` du site, `apache2ctl configtest`, `reload` (le port 443 s'ajoute tout seul à ports.conf quand ssl est activé)
- `curl -k https://localhost` → ignore la vérification du certificat (nécessaire avec un certificat auto-signé) ; `curl -vk https://localhost` pour voir le certificat présenté
- Le bloc du port 80 redirige tout vers HTTPS (module `alias`, actif par défaut)

## Journaux (logs)

- `ErrorLog ${APACHE_LOG_DIR}/example-error.log` → erreurs du serveur ; `LogLevel warn` → verbosité (debug, info, notice, warn, error, crit, alert, emerg) ; `LogLevel info ssl:warn` → un niveau par module
- `CustomLog ${APACHE_LOG_DIR}/example-access.log combined` → journal des accès avec un format NOMMÉ. Formats prêts à l'emploi : `common` (sans navigateur ni page d'origine) et `combined` (avec). Ajouter un format : `LogFormat "%h %t \"%r\" %>s %b" mini`
- Champs : `%h` IP du client / `%t` date / `%r` ligne de requête (« GET /page HTTP/1.1 ») / `%>s` code HTTP final / `%b` octets envoyés / `%{User-Agent}i` navigateur / `%D` durée en microsecondes
- Un VirtualHost sans `ErrorLog` / `CustomLog` écrit dans les journaux du serveur principal ; en définir par site (comme ci-dessus) sépare les journaux de chaque site
- Rotation : `/etc/logrotate.d/apache2` est fourni par le paquet (voir [Logs](logs.md)) et recharge Apache en douceur après la rotation
- Analyser (voir [Texte](texte.md)) : `tail -f /var/log/apache2/access.log` en direct ; `awk '{print $9}' /var/log/apache2/access.log | sort | uniq -c` → nombre de réponses par code HTTP ; `grep ' 404 ' /var/log/apache2/access.log` → les pages introuvables

## Reverse proxy

Équivalent du reverse proxy nginx (voir [Nginx](nginx.md)) : Apache transmet les requêtes à un autre serveur.

- `sudo a2enmod proxy proxy_http` → active les modules
- Dans le VirtualHost : `ProxyPass / http://127.0.0.1:8080/` (transmet) et `ProxyPassReverse / http://127.0.0.1:8080/` (réécrit les redirections renvoyées par le serveur cible) ; `ProxyPreserveHost On` garde le nom d'hôte demandé par le client
- Sous SELinux, Apache n'a pas le droit d'ouvrir des connexions réseau : activer le booléen `httpd_can_network_connect` (voir [SELinux / AppArmor](selinux-apparmor.md))

## Dépannage

- Site qui ne répond pas : `apache2ctl configtest`, puis `journalctl -xeu apache2` (voir [Logs](logs.md)), puis `error.log` ; vérifier qui écoute le port avec `ss -ltunp` (voir [Réseau](reseau.md))
- Erreur 403 : permissions (le dossier ET ses parents doivent être traversables par www-data), `Require all granted` manquant, ou badge SELinux incorrect (RHEL : `httpd_sys_content_t`, voir [SELinux / AppArmor](selinux-apparmor.md))
- Erreur 404 : mauvais `DocumentRoot` ou fichier absent ; erreur 500 : lire `error.log` (souvent un `.htaccess` invalide ou un module manquant)
- Mauvais site affiché : `apache2ctl -S` et l'ordre des `ServerName`
