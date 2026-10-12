# Modules du noyau

[← Index](../README.md)

Module = morceau de noyau (pilote, système de fichiers, fonction réseau) qu'on charge / décharge sans redémarrer. Fichiers dans `/lib/modules/$(uname -r)/`.

## Charger et décharger

- `lsmod` → modules actuellement chargés (colonne Used by = dépendances) ; `lsmod | grep kvm` pour filtrer
- `modinfo br_netfilter` → description, dépendances et paramètres d'un module
- `sudo modprobe br_netfilter` → charge le module ET ses dépendances
- `sudo modprobe -r br_netfilter` → décharge le module (refusé s'il est utilisé)
- `sudo modprobe -n -v br_netfilter` → simulation : montre ce qui serait chargé, sans rien faire
- `sudo modprobe usbcore autosuspend=5` → charge avec un paramètre
- `sudo insmod /chemin/module.ko` / `sudo rmmod module` → versions bas niveau, SANS gestion des dépendances (préférer modprobe)
- `sudo depmod -a` → recalcule les dépendances entre modules
- `cat /sys/module/NOM/parameters/PARAM` → valeur actuelle d'un paramètre d'un module chargé

## Persistant au démarrage

- `/etc/modules-load.d/*.conf` → un nom de module par ligne, chargé à chaque démarrage. Exemple : `echo "br_netfilter" | sudo tee /etc/modules-load.d/k8s.conf`
- `/etc/modprobe.d/*.conf` → paramètres et règles de chargement :
  - `options usbcore autosuspend=5` → paramètre appliqué à chaque chargement du module
  - `blacklist nouveau` → empêche le chargement AUTOMATIQUE (un modprobe manuel reste possible)
  - `install nouveau /bin/false` → interdit le chargement même manuel
  - `alias` → nom alternatif d'un module
- Si le module est utile très tôt au démarrage : régénérer l'initramfs (voir [GRUB et récupération](grub-recuperation.md))

## Paramètres du noyau (sysctl)

Les réglages du noyau en marche (`net.ipv4.ip_forward`, etc.) ne sont pas des modules : voir [Système](systeme.md).
