# SELinux / AppArmor

[← Index](../README.md)

## SELinux : principe

À quoi ça sert : une 2e couche de sécurité EN PLUS des permissions rwx. Même si rwx autorise l'accès, SELinux peut le refuser → un programme piraté (ex : Apache) ne peut pas sortir de son périmètre.

Image : chaque processus et chaque fichier porte un BADGE (= contexte, ou étiquette). Une règle dit quel badge de processus a le droit d'accéder à quel badge de fichier. Ex : Apache (badge httpd_t) peut lire les fichiers httpd_sys_content_t, mais pas /etc/shadow (badge shadow_t).

Dans un contexte comme `system_u:object_r:httpd_sys_content_t:s0`, la partie importante = le TYPE (celle qui finit par _t).

- `ls -Z` / `ps -eZ` (ou `ps axZ`) / `id -Z` → affiche le badge des fichiers / des processus / de l'utilisateur

## Les 3 modes

- Enforcing → bloque ce qui n'est pas autorisé + le note dans les logs
- Permissive → ne bloque RIEN, note seulement ce qu'il aurait bloqué (pratique pour diagnostiquer)
- Disabled → SELinux éteint

Commandes :

- `getenforce` → affiche le mode actuel / `sestatus` → état détaillé
- `sudo setenforce 0` (Permissive) / `sudo setenforce 1` (Enforcing) → TEMPORAIRE, jusqu'au prochain redémarrage
- Permanent : ligne `SELINUX=enforcing` (ou permissive / disabled) dans `/etc/selinux/config`

## Étiquettes (badges) des fichiers

- `sudo restorecon -v /usr/bin/less` → « restore the DEFAULT label » d'un fichier dont le badge a été modifié : remet le badge prévu par le registre. Pas besoin de semanage pour un fichier système (le registre connaît déjà le bon label). Vérifier avant/après avec `ls -Z /usr/bin/less`

- `chcon -t httpd_sys_content_t /web/index.html` → TEMPORAIRE : change le badge tout de suite (= autocollant sur la porte), mais restorecon ou un ré-étiquetage du système l'annule
- `sudo semanage fcontext -a -t httpd_sys_content_t '/web(/.*)?'` → PERMANENT, étape 1 : inscrit la règle dans le registre officiel (-a = add ; '/web(/.*)?' = /web et tout son contenu)
- `sudo restorecon -Rv /web` → PERMANENT, étape 2 : applique le registre aux fichiers (-R = récursif, -v = affiche les changements). ⚠️ semanage seul ne change rien aux fichiers existants : TOUJOURS les deux étapes

- `sudo semanage fcontext -a -e /var/www /web` puis `sudo restorecon -Rv /web` → plus court : /web reçoit les mêmes badges que /var/www (équivalence)
- `sudo touch /.autorelabel` puis redémarrer → ré-étiquette TOUT le système au prochain démarrage (long) ; utile après avoir désactivé SELinux un moment ou restauré des fichiers

## Ports

Un service ne peut écouter que sur les ports de son type SELinux : nginx sur le port 8888 est refusé tant que le port n'a pas le bon badge.

- `sudo semanage port -l | grep http` → ports autorisés par type (http_port_t : 80, 443, 8008, 8080…)
- `sudo semanage port -a -t http_port_t -p tcp 8888` → autorise nginx / Apache à écouter sur 8888 (-a ajouter ; -m modifier si le port a déjà un type). Pour SSH : `-t ssh_port_t`

## Comprendre un refus

- `sudo ausearch -m AVC -ts recent` → derniers refus SELinux (messages AVC) lus dans /var/log/audit/audit.log ; `-ts today` pour la journée
- Méthode : 1) `setenforce 0` pour tester : si ça marche, c'est bien SELinux ; 2) lire les refus AVC ; 3) corriger le badge (semanage fcontext + restorecon), le booléen (setsebool -P) ou le port (semanage port) ; 4) `setenforce 1`

## Booléens

Booléens = interrupteurs on/off prêts à l'emploi pour les autorisations courantes.

- `getsebool -a` → liste tous les interrupteurs et leur état (`getsebool -a | grep httpd` pour filtrer)
- `sudo setsebool -P httpd_can_network_connect on` → autorise Apache à se connecter au réseau ; -P = PERMANENT (sans -P, l'interrupteur revient à sa position au redémarrage)

## Dernier recours

Si aucun badge ni interrupteur ne règle le problème :

- `sudo audit2allow -a -M mymodule` → lit les refus de SELinux dans les logs et fabrique une règle qui les autorise (crée le fichier mymodule.pp)
- `sudo semodule -i mymodule.pp` → installe cette règle (-i = install)

## AppArmor

AppArmor = même but (couche de sécurité en plus de rwx) mais autre méthode : au lieu de badges sur les fichiers, chaque PROGRAMME a un profil qui liste les chemins auxquels il a droit. C'est le système par défaut sur Ubuntu.

- `sudo aa-status` → liste les profils AppArmor chargés et leur mode (enforce = bloque, complain = note seulement)
- `/etc/apparmor.d/` → les profils (un fichier par programme, ex : `usr.sbin.nginx`) ; `/etc/apparmor.d/local/usr.sbin.nginx` → ajouts locaux à un profil sans toucher au fichier d'origine, par exemple une ligne `/web/** r,`
- `sudo aa-complain /etc/apparmor.d/usr.sbin.nginx` → passe le profil en mode complain (note sans bloquer : pour diagnostiquer) ; `sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx` → remet en enforce ; `sudo aa-disable /etc/apparmor.d/usr.sbin.nginx` → désactive le profil
- `sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx` → recharge un profil après modification
- `dmesg | grep -i 'apparmor.*denied'` → les refus (DENIED) enregistrés ; `sudo journalctl -k | grep -i apparmor` → idem via le journal
- `sudo aa-genprof /usr/sbin/monprog` → crée un profil en observant le programme (outils du paquet apparmor-utils) ; `sudo aa-logprof` → propose d'adapter les profils d'après les refus enregistrés
- `ps auxZ` → affiche le profil AppArmor de chaque processus ; `sudo aa-unconfined` → processus en écoute réseau SANS profil
