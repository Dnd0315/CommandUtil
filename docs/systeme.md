# Système

[← Index](../README.md)

## Ressources et matériel

- `df -h` → espace libre/utilisé par système de fichiers (-h = lisible : Go, Mo)
- `du -sh dir/` → taille totale d'un dossier (-s = total seulement)
- `free -h` → mémoire RAM et swap utilisées/libres
- `uptime` → depuis quand la machine tourne + charge moyenne
- `lscpu` → infos processeur / `lspci` → matériel PCI (cartes réseau, graphiques…)

## Noyau et identité de la machine

- `uname -r` → version (release) du noyau Linux, ex : 5.15.0-69-generic ; `uname -a` → tout
- `sudo hostnamectl set-hostname srv1` → change le nom de la machine (persistant)

## sysctl (paramètres du noyau)

- `sysctl -a` → liste tous les paramètres du noyau
- `sysctl -n net.ipv4.ip_forward` → seulement la valeur
- `cat /proc/sys/net/ipv4/ip_forward` = `sysctl net.ipv4.ip_forward` → valeur actuelle d'un paramètre noyau (/proc/sys/ = les paramètres sysctl, points → barres)
- `sudo sysctl -w net.ipv4.ip_forward=1` → modifie un paramètre (perdu au redémarrage)
- Persistant : `echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-custom.conf` puis `sudo sysctl --system` (ou `sysctl -p fichier`)
