# Nginx (reverse proxy)

[← Index](../README.md)

Équivalent Apache (hôtes virtuels, accès restreint, HTTPS, reverse proxy) : voir [Apache](apache.md).

Reverse proxy = nginx reçoit les requêtes des clients et les TRANSMET à un autre serveur (ex : google.com).

Dossiers :

- `/etc/nginx/sites-available/` = toutes les configs disponibles
- `/etc/nginx/sites-enabled/` = liens symboliques vers celles qui sont ACTIVES (nginx ne lit que celui-ci)

## Mettre en place un proxy

1. Désactiver le site par défaut : `sudo rm /etc/nginx/sites-enabled/default` → supprime seulement le LIEN, le fichier d'origine reste dans sites-available
2. Éditer la config du proxy : `sudo vi /etc/nginx/sites-available/proxy.conf` (chaque ligne finit par `;` sauf les accolades)

   ```nginx
   server {
       listen 80;
       location / {
           proxy_pass http://google.com;
           include proxy_params;
       }
   }
   ```

3. Activer avec un lien symbolique (chemins ABSOLUS) : `sudo ln -s /etc/nginx/sites-available/proxy.conf /etc/nginx/sites-enabled/proxy.conf`
4. Tester la syntaxe AVANT de recharger : `sudo nginx -t` → en cas d'échec (« Job for nginx.service failed »), affiche l'erreur [emerg] exacte (fichier et ligne fautifs). Corriger le fichier indiqué, puis recommencer `nginx -t`
5. Appliquer avec `reload` (voir [systemd](systemd.md)) ; si le service ne démarre pas, détails avec `systemctl status` et `journalctl -xeu` (voir [Logs](logs.md))

## Vérifier

- `curl -I localhost` (ou `curl localhost`) → la réponse vient du serveur cible (google.com)

## Astuce vi

Si vi affiche « Error detected while processing /usr/share/vim/vimrc … E484 … Press ENTER » : message SANS gravité (fichiers de coloration manquants), appuyer sur Entrée, l'éditeur fonctionne ; sinon utiliser `nano`.
