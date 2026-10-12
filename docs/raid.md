# RAID logiciel (mdadm)

[← Index](../README.md)

Niveaux à connaître :

- RAID 0 → agrégat sans redondance (≥ 2 disques) : plus de vitesse, UNE panne = tout est perdu
- RAID 1 → miroir (≥ 2 disques) : les données sont copiées sur chaque disque, survit tant qu'il reste un disque
- RAID 5 → parité répartie (≥ 3 disques) : tolère 1 panne ; capacité = (n - 1) disques
- RAID 6 → double parité (≥ 4 disques) : tolère 2 pannes
- RAID 10 → miroirs agrégés (≥ 4 disques) : vitesse + redondance

## Créer un RAID

- `sudo apt install mdadm` → installation
- `sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc` → crée /dev/md0 en RAID 1 avec ces deux disques (ou partitions, ex : /dev/sdb1)
- `sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 --spare-devices=1 /dev/sdb /dev/sdc /dev/sdd /dev/sde` → RAID 5 de 3 disques + 1 disque de secours (spare) qui prend le relais en cas de panne
- `cat /proc/mdstat` → état en direct et progression de la synchronisation ; [UU] = tous les disques OK, [U_] = un disque manquant ou en panne
- `sudo mdadm --detail /dev/md0` → détails : niveau, état (clean, degraded), disques actifs / spare / failed
- `sudo mdadm --examine /dev/sdb` → métadonnées RAID lues sur un disque membre

Ensuite, /dev/md0 se formate et se monte comme une partition ordinaire (`mkfs`, `mount`, fstab : voir [Storage](storage.md)).

## Rendre persistant

- `sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf` → enregistre la définition du tableau (RHEL : /etc/mdadm.conf). Sans cela, le nom peut devenir /dev/md127 au redémarrage
- Debian / Ubuntu : régénérer ensuite l'initramfs (voir [GRUB et récupération](grub-recuperation.md))
- fstab : utiliser l'UUID du tableau (`blkid /dev/md0`), comme pour une partition

## Panne et remplacement d'un disque

- `sudo mdadm /dev/md0 --fail /dev/sdb` → simule / déclare la panne d'un disque
- `sudo mdadm /dev/md0 --remove /dev/sdb` → retire le disque défaillant du tableau
- `sudo mdadm /dev/md0 --add /dev/sdd` → ajoute le disque de remplacement : la reconstruction démarre toute seule (suivre avec `cat /proc/mdstat`). Ajouté à un tableau sain, il devient spare

## Agrandir, arrêter, supprimer

- `sudo mdadm --grow /dev/md0 --raid-devices=4` → après avoir ajouté un disque (`--add`), l'intègre au RAID 5 ; agrandir ensuite le système de fichiers (voir [Storage](storage.md))
- `sudo mdadm --stop /dev/md0` → arrête le tableau (démonter avant)
- `sudo mdadm --assemble --scan` → réassemble les tableaux connus ; `sudo mdadm --assemble /dev/md0 /dev/sdb /dev/sdc` → réassemble explicitement
- `sudo mdadm --zero-superblock /dev/sdb /dev/sdc` → après `--stop`, efface les métadonnées RAID des disques pour les réutiliser (retirer aussi la ligne de mdadm.conf et de fstab)
