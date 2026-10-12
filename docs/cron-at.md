# cron / at (tâches planifiées)

[← Index](../README.md)

## cron

Syntaxe d'une ligne cron : `minute heure jour_du_mois mois jour_de_la_semaine commande` (\* = tous ; 0 ou 7 = dimanche)

- `30 2 * * 1-5 /script.sh` → à 2h30, du lundi au vendredi
- `*/15 * * * * /script.sh` → toutes les 15 minutes
- `0 0 1 * * /script.sh` → à minuit, le 1er de chaque mois
- `@reboot /script.sh` → au démarrage de la machine ; autres raccourcis : `@hourly`, `@daily`, `@weekly`, `@monthly`
- `0 3 * * * /script.sh >> /var/log/job.log 2>&1` → cron envoie la sortie par mail à l'utilisateur : la rediriger dans un fichier (voir [Créer / écrire dans un fichier](fichiers-et-redirections.md)) ; `MAILTO=""` en haut de la crontab supprime le courrier
- `PATH=/usr/local/bin:/usr/bin:/bin` (en haut de la crontab) → cron a un PATH minimal : donner les chemins complets ou définir PATH
- `cat /etc/crontab` → crontab système ; elle a un champ UTILISATEUR en plus avant la commande
- `crontab -e` → édite la crontab de l'utilisateur courant
- `crontab -l` → affiche la crontab / `crontab -r` → la supprime entièrement
- `sudo crontab -e -u john` → édite la crontab de john

## Dossiers cron

- `/etc/cron.hourly/`, `/etc/cron.daily/`, `/etc/cron.weekly/`, `/etc/cron.monthly/` → déposer un SCRIPT exécutable (chmod +x) ; le nom ne doit pas contenir de point (`backup`, pas `backup.sh`) sinon run-parts l'ignore
- `/etc/cron.d/` → fichiers au format crontab SYSTÈME (avec le champ utilisateur) : le bon endroit pour une tâche installée par un paquet ou un déploiement
- `/var/spool/cron/crontabs/` → crontabs des utilisateurs (RHEL : `/var/spool/cron/`) ; ne pas les éditer à la main, utiliser `crontab -e`
- `/etc/cron.allow` / `/etc/cron.deny` → qui a le droit d'utiliser crontab : si cron.allow existe, SEULS les utilisateurs listés peuvent ; sinon cron.deny interdit ceux qui y sont listés (même principe avec `/etc/at.allow` et `/etc/at.deny` pour at)

## Vérifier qu'une tâche s'est exécutée

- `journalctl -u cron` → exécutions de cron (RHEL : service `crond`, fichier `/var/log/cron`)
- `grep CRON /var/log/syslog` → mêmes lignes dans le fichier de log (Debian / Ubuntu)
- Sinon, regarder l'effet de la tâche (fichier créé, log applicatif) ou le fichier où on a redirigé la sortie

## at (une seule fois)

- `at 15:00` / `at now + 30 minutes` → tâche exécutée UNE fois ; taper la commande puis Ctrl+D pour valider
- `at -f /chemin/script.sh 02:00 tomorrow` → exécute un script fichier plutôt qu'une commande tapée
- `atq` → liste les tâches at en attente / `atrm 3` → supprime la tâche n°3
- `at -c 3` → affiche le contenu de la tâche n°3 (environnement + commande)

## Timers systemd

Alternative moderne à cron : un fichier `.timer` déclenche un fichier `.service` du même nom. Avantages : logs dans journalctl, dépendances, rattrapage des exécutions manquées.

- `systemctl list-timers` → timers actifs, prochaine exécution et dernière ; `--all` ajoute les timers inactifs
- Le service à lancer, `/etc/systemd/system/backup.service` :

  ```ini
  [Unit]
  Description=Sauvegarde

  [Service]
  Type=oneshot
  ExecStart=/usr/local/bin/backup.sh
  ```

- Le timer, `/etc/systemd/system/backup.timer` :

  ```ini
  [Unit]
  Description=Sauvegarde quotidienne

  [Timer]
  OnCalendar=*-*-* 02:30:00
  Persistent=true

  [Install]
  WantedBy=timers.target
  ```

- Puis `daemon-reload` et `enable --now backup.timer` (voir [systemd](systemd.md)) : activer le TIMER, pas le service
- `OnCalendar=` → date absolue : `daily`, `weekly`, `Mon..Fri 08:00`, `*:0/15` (toutes les 15 minutes) ; `OnBootSec=5min` (5 min après le démarrage) et `OnUnitActiveSec=1h` (1 h après la dernière exécution) → délais relatifs
- `Persistent=true` → si la machine était éteinte à l'heure prévue, la tâche est lancée au démarrage suivant
- `systemd-analyze calendar "Mon..Fri 08:00"` → vérifie une expression OnCalendar et affiche la prochaine échéance
- Tester la tâche tout de suite : démarrer le SERVICE (pas le timer) ; ses logs : `journalctl -u backup.service`
