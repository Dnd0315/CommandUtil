# Time

[← Index](../README.md)

## Heure et fuseau horaire

- `timedatectl` → heure, fuseau horaire, état de la synchronisation NTP (donne aussi le fuseau du système)
- `timedatectl list-timezones` → liste des fuseaux disponibles
- `sudo timedatectl set-timezone Asia/Dubai` → change le fuseau horaire
- `date +%Z` → abréviation du fuseau (UTC) ; `cat /etc/timezone` → fuseau horaire du système

## Régler l'heure à la main

- `sudo timedatectl set-ntp false` puis `sudo timedatectl set-time "2026-10-12 14:30:00"` → fixe l'heure (impossible tant que la synchronisation NTP est active)
- `sudo date -s "2026-10-12 14:30:00"` → équivalent, avec la commande date
- `sudo hwclock --show` → affiche l'horloge matérielle (RTC)
- `sudo hwclock --systohc` → copie l'heure du système vers l'horloge matérielle (`--hctosys` : sens inverse)

## NTP avec systemd-timesyncd

- `sudo timedatectl set-ntp true` → active la synchronisation automatique de l'heure
- Choisir les serveurs NTP (systemd-timesyncd) : éditer `/etc/systemd/timesyncd.conf`, section [Time] → `NTP=0.europe.pool.ntp.org 1.europe.pool.ntp.org` (⚠️ retirer le # devant NTP= ; serveurs séparés par des ESPACES)
- `sudo systemctl restart systemd-timesyncd` → applique la config
- `timedatectl timesync-status` / `timedatectl show-timesync` → vérifie quels serveurs NTP sont utilisés
- `man timesyncd.conf` → liste toutes ces options (chercher avec /Fallback)
- `ntpdate -q 0.pool.ntp.org` → teste un serveur NTP sans changer l'heure
- `sudo service systemd-timesyncd status` → vérifier qu'il n'y a pas d'erreur
- `grep systemd-timesyncd /var/log/syslog` → les logs montrent le serveur utilisé

## NTP avec chrony

chrony remplace timesyncd (installé par défaut sur RHEL ; sur Ubuntu : `sudo apt install chrony`, timesyncd se désactive alors tout seul). Il sait aussi servir l'heure aux autres machines.

- `/etc/chrony/chrony.conf` (RHEL : `/etc/chrony.conf`) → serveurs : `server 0.pool.ntp.org iburst` ou `pool 2.pool.ntp.org iburst` (iburst = synchronisation rapide au démarrage)
- `sudo systemctl restart chrony` → applique la config (service `chronyd` sur RHEL)
- `chronyc sources -v` → serveurs utilisés ; `^*` = serveur actuellement suivi, `^+` = serveur acceptable
- `chronyc tracking` → décalage actuel, stratum, dernière correction
- `sudo chronyc makestep` → force un rattrapage immédiat de l'heure
- `allow 10.0.0.0/24` (dans chrony.conf) → cette machine sert l'heure à ce réseau ; `local stratum 10` la fait servir même sans accès à Internet
