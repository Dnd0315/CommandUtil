# Logs

[← Index](../README.md)

## Fichiers de logs

- `ls /var/log` → dossier des logs texte
- `grep -r 'ssh' /var/log` → cherche « ssh » dans tous les logs
- `which sudo` → donne le chemin complet d'une commande (/usr/bin/sudo)
- Principaux fichiers (Debian / Ubuntu ; RHEL entre parenthèses) :
  - `/var/log/syslog` (RHEL : `/var/log/messages`) → journal général du système
  - `/var/log/auth.log` (RHEL : `/var/log/secure`) → connexions, sudo, erreurs PAM, SSH
  - `/var/log/kern.log` → messages du noyau
  - `/var/log/dpkg.log` et `/var/log/apt/history.log` → paquets installés / supprimés (RHEL : `dnf history`)
  - `/var/log/cron` (RHEL) → exécutions cron
- Les fichiers tournés (`syslog.1`, `syslog.2.gz`) se lisent sans décompresser avec `zcat` / `zgrep` (voir [Archives](archives.md))

## Messages du noyau

- `dmesg` → messages du noyau depuis le démarrage (matériel, disques, réseau, erreurs)
- `dmesg -T` → avec des dates lisibles ; `dmesg -w` → suit en direct
- `sudo dmesg -l err,warn` → seulement ces niveaux de gravité
- `journalctl -k` → mêmes messages noyau via le journal (combinable avec `-b -1`)

## journalctl

- `journalctl` → tous les logs systemd (du plus ancien au plus récent)
- `journalctl /usr/bin/sudo` → logs générés par ce programme (chemin complet obligatoire, d'où `which`)
- `journalctl -u ssh.service` → logs d'un service précis
- `sudo journalctl -xeu nginx.service` → logs d'un service avec explications (-x), directement à la fin (-e), pour l'unité (-u) : le réflexe quand un service refuse de démarrer
- `journalctl -e` → saute directement à la fin
- `journalctl -f` → suit les nouveaux logs en direct
- `journalctl -p err` → seulement les erreurs et plus grave (info, warning, err, crit…)
- `journalctl -b 0` → logs du démarrage actuel / `-b -1` → du démarrage précédent / `journalctl --list-boots` → liste des démarrages enregistrés
- `journalctl --since "1 hour ago"` → logs de la dernière heure (accepte aussi "2026-10-01 10:00") ; `--until "2026-10-01 12:00"` fixe la fin de la plage
- `journalctl --disk-usage` → place occupée par le journal
- `sudo journalctl --vacuum-time=7d` → supprime les entrées de plus de 7 jours ; `--vacuum-size=200M` → limite la taille totale
- Journal PERSISTANT (par défaut, il disparaît au redémarrage sur certaines distributions) : `sudo mkdir -p /var/log/journal` puis `sudo systemctl restart systemd-journald`, ou `Storage=persistent` dans `/etc/systemd/journald.conf`

## rsyslog

rsyslog écrit les messages dans les fichiers de /var/log selon des règles : `/etc/rsyslog.conf` et `/etc/rsyslog.d/*.conf`.

- Format d'une règle : `facility.priorité  destination`. Facilities : auth, authpriv, cron, daemon, kern, mail, user, local0 à local7. Priorités : debug, info, notice, warning, err, crit, alert, emerg
- `auth,authpriv.*  /var/log/auth.log` → tout ce qui vient de auth / authpriv dans ce fichier
- `*.info;mail.none  /var/log/messages` → priorité info et plus pour tout, sauf le courrier
- `local0.*  /var/log/monapp.log` → journal dédié à une application qui logue sur local0
- `*.*  @@loghost:514` → envoie tout à un serveur de logs en TCP (`@` simple = UDP)
- `sudo systemctl restart rsyslog` → applique la config
- `logger -p local0.info "message de test"` → écrit un message de test avec cette facility / priorité ; `logger -t montag "texte"` → avec une étiquette

## logrotate (rotation des logs)

- `/etc/logrotate.conf` → réglages généraux ; `/etc/logrotate.d/*` → un fichier par application. Exécuté chaque jour (cron ou timer)
- Exemple de `/etc/logrotate.d/monapp` :

  ```
  /var/log/monapp/*.log {
      weekly
      rotate 4
      compress
      delaycompress
      missingok
      notifempty
      create 0640 root adm
  }
  ```

- `weekly` / `daily` / `monthly` → fréquence ; `rotate 4` → garde 4 anciens fichiers ; `compress` → gzip des anciens ; `delaycompress` → ne compresse pas le plus récent ; `missingok` → pas d'erreur si le log est absent ; `notifempty` → pas de rotation d'un fichier vide ; `create 0640 root adm` → droits du nouveau fichier ; `size 100M` → rotation selon la taille
- `sudo logrotate -d /etc/logrotate.d/monapp` → simulation : affiche ce qui serait fait, sans rien changer
- `sudo logrotate -f /etc/logrotate.d/monapp` → force la rotation maintenant
