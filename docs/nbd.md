# NBD (disque réseau : Network Block Device)

[← Index](../README.md)

NBD = un disque situé sur un serveur, présenté sur notre machine comme un disque local (/dev/nbd0, /dev/nbd1…). Outil : `nbd-client` (`sudo apt install nbd-client`). Un serveur peut proposer plusieurs « exports » (disk1, disk2…).

- Repérer le disque nbd0 et son point de montage (ex : /share) avec `lsblk` (voir [Storage](storage.md))
- `sudo umount /share` → démonte d'abord (TOUJOURS démonter AVANT de déconnecter)
- `sudo nbd-client -d /dev/nbd0` → déconnecte le disque réseau (-d = disconnect)
- `sudo nbd-client -l 127.0.0.1` → liste les exports proposés par le serveur NBD (-l = list) : repérer le NOM du 2e export
- `sudo nbd-client 127.0.0.1 -N disk2` → attache l'export disk2 (-N = nom de l'export) ; il réapparaît comme /dev/nbd0
- `sudo mount /dev/nbd0 /mnt` → monte-le sur /mnt ; vérifier avec `lsblk` (la colonne MOUNTPOINTS doit afficher /mnt)
