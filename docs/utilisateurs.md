# Utilisateurs

[← Index](../README.md)

## Comptes

- `sudo useradd -m -s /bin/bash -G sudo john` → crée john avec un home (-m), un shell bash (-s) et le groupe secondaire sudo (-G)
- `sudo useradd -u 1500 -g dev -c "John Doe" -d /home/john -m john` → fixe l'UID (-u), le groupe primaire (-g), le commentaire / nom complet (-c) et le home (-d)
- `sudo useradd -r -s /usr/sbin/nologin svc_app` → compte SYSTÈME (-r : UID bas, pas de home) qui ne peut pas ouvrir de session (nologin) : pour un service
- `sudo adduser john` → version interactive (Debian/Ubuntu) qui crée tout et demande le mot de passe
- `sudo passwd john` → définit / change le mot de passe de john
- `sudo passwd -l john` / `-u` → verrouille / déverrouille le mot de passe ; `sudo passwd -e john` → l'expire (à changer à la prochaine connexion) ; `passwd -S john` → état du mot de passe (L verrouillé, P actif, NP aucun)
- `sudo userdel -r john` → supprime john et son home (-r) ; `deluser` = équivalent Ubuntu
- `sudo usermod -aG docker john` → AJOUTE john au groupe docker (⚠️ -G sans -a remplace tous ses groupes secondaires)
- `sudo usermod -g dev john` → change le groupe PRIMAIRE de john
- `sudo usermod -s /bin/zsh john` → change le shell
- `sudo usermod -d /new/home -m john` → change le home et y déplace le contenu (-m)
- `sudo usermod -L john` / `-U` → verrouille / déverrouille le compte
- `sudo usermod -e 2026-12-31 john` → date d'expiration du compte
- `id john` → UID, GID et groupes de john / `whoami` → utilisateur courant
- `/etc/passwd` → liste des comptes (nom, UID, GID, home, shell) / `/etc/shadow` → mots de passe chiffrés / `/etc/group` → groupes

## Expiration du mot de passe

- `sudo chage -l john` → affiche la politique d'expiration du mot de passe
- `sudo chage -M 90 john` → mot de passe à changer tous les 90 jours max
- `sudo chage -m 1 -W 7 -I 5 john` → délai minimum entre deux changements (-m, en jours), préavis avant expiration (-W), jours d'inactivité tolérés après expiration avant blocage du compte (-I)
- `sudo chage -E 2026-12-31 john` → le compte expire à cette date
- `sudo chage -d 0 john` → oblige john à changer son mot de passe à la prochaine connexion

## Groupes

- `sudo groupadd dev` → crée un groupe / `sudo groupdel dev` → le supprime
- `sudo gpasswd --add john dev` → ajoute john au groupe dev / `--delete` → l'en retire
- `sudo groupmod -n devops dev` → renomme le groupe dev en devops
- `newgrp dev` → ouvre un shell où dev est le groupe primaire effectif (il faut en être membre) ; `exit` pour revenir

## sudo

- `sudo visudo` → édite /etc/sudoers en vérifiant la syntaxe (ne jamais l'éditer directement)
- `sudo visudo -f /etc/sudoers.d/john` → un fichier par utilisateur / règle dans /etc/sudoers.d/ (plus propre que de toucher /etc/sudoers ; visudo gère la syntaxe et les droits 440)
- `sudo visudo -c` → vérifie la syntaxe de sudoers et de sudoers.d sans rien modifier
- Lignes sudoers :
  - `john ALL=(ALL) ALL` → john peut tout faire avec sudo
  - `%dev ALL=(ALL) NOPASSWD: ALL` → idem pour le groupe dev, sans mot de passe
  - `john ALL=(ALL) /usr/bin/systemctl` → restreint john à cette seule commande
  - `Cmnd_Alias SERVICES = /usr/bin/systemctl restart nginx, /usr/bin/systemctl status nginx` puis `%dev ALL=(ALL) NOPASSWD: SERVICES` → un alias regroupe plusieurs commandes (`User_Alias` fait de même pour des utilisateurs)
  - Groupe administrateur par défaut : `%sudo` (Debian / Ubuntu) ou `%wheel` (RHEL)
- Autoriser UNE seule commande, sans mot de passe : `user2 ALL=(root) NOPASSWD: /bin/bash /root/dangerous.sh` → user2 peut lancer exactement `sudo bash /root/dangerous.sh` et rien d'autre

## Devenir un autre utilisateur (root)

- `su - john` → shell de LOGIN complet de john (son environnement, son home) : demande le mot de passe de john ; `su john` garde l'environnement actuel ; `su -` = `su - root` : vrai shell root (mot de passe ROOT)
- `su -c 'id' john` → exécute une seule commande en tant que john
- `sudo -i` → shell root de login avec TON mot de passe (pas besoin de connaître celui de root) ; `sudo -s` → shell root en gardant l'environnement ; `sudo su -` = équivalent historique
- `sudo -u john commande` → exécute la commande en tant que john ; `sudo -u john -i` → shell de john
- `sudo -l` → liste ce que l'utilisateur courant a le droit de faire avec sudo ; `sudo -l -U john` → pour john
- `exit` → revient à l'utilisateur précédent
- Sur Ubuntu, root est verrouillé par défaut : `sudo passwd root` lui donne un mot de passe, `sudo passwd -l root` le reverrouille

## Connexions

- `who` → utilisateurs connectés ; `w` → idem avec la commande en cours et la charge de la machine
- `last` → historique des connexions (/var/log/wtmp) ; `last -n 10` → les 10 dernières
- `sudo lastb` → connexions ÉCHOUÉES (/var/log/btmp) / `lastlog` → date de dernière connexion de chaque compte
- `sudo pkill -KILL -u john` → termine tous les processus de john (le déconnecte)

## Valeurs par défaut des nouveaux comptes

- `/etc/login.defs` → paramètres des comptes créés ensuite : PASS_MAX_DAYS / PASS_MIN_DAYS / PASS_WARN_AGE (expiration), UID_MIN / UID_MAX (plages d'UID), UMASK, USERGROUPS_ENAB (un groupe privé par utilisateur), ENCRYPT_METHOD. Ne change pas les comptes existants
- `useradd -D` → affiche les valeurs par défaut de useradd (fichier /etc/default/useradd : SHELL, HOME, INACTIVE, EXPIRE, SKEL) ; `sudo useradd -D -s /bin/bash` → change le shell par défaut des futurs comptes
- `/etc/skel/` → modèle copié dans le home de tout NOUVEAU compte (.bashrc, .profile…) : y déposer un fichier = tous les futurs utilisateurs l'auront (les comptes existants ne changent pas). Voir aussi [Variables d'environnement](variables-environnement.md)
- `/etc/adduser.conf` → équivalent de login.defs / useradd -D pour `adduser` (Debian / Ubuntu)

## Limites de ressources (ulimit)

Les limites de /etc/security/limits.conf sont appliquées par le module PAM pam_limits : voir [PAM](pam.md). Quotas d'espace disque : voir [Quotas](quotas.md).

- `ulimit -a` → affiche les limites de ressources de la session (fichiers ouverts, processus…)
- `ulimit -u` → affiche la limite de processus actuelle
- `ulimit -S -u 1100` → tente de la changer en soft (échoue avec « cannot modify limit » si la hard est plus basse)
- `ulimit -H -u` → affiche la hard. Tester avec `su jackie` puis `ulimit -u`
- Persistant : `/etc/security/limits.conf` → `john hard nproc 50` (john max 50 processus ; soft = limite par défaut modifiable, hard = plafond ; nofile = fichiers ouverts ; @groupe pour un groupe)
- `@operators hard maxlogins 1` → le groupe operators ne peut avoir qu'UNE session ouverte à la fois (dans /etc/security/limits.conf, ligne par ligne : domaine / type / item / valeur). Non testable en simulateur
