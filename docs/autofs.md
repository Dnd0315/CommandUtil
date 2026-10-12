# Montage à la demande (autofs)

[← Index](../README.md)

Un montage « à la demande » n'est effectué que lorsqu'on accède au dossier, puis démonté après un délai d'inactivité : utile pour NFS, les homes, les disques amovibles.

## autofs

- `sudo apt install autofs` → installation (activer le service : voir [systemd](systemd.md))
- `/etc/auto.master` → liste des dossiers gérés : `/mnt/auto /etc/auto.nfs --timeout=60` = dossier parent / fichier de correspondances (map) / démontage après 60 s d'inactivité. Sur Ubuntu, les fichiers de `/etc/auto.master.d/*.autofs` sont aussi lus
- `/etc/auto.nfs` (la map) → `clé  options  source`. Exemple : `share -fstype=nfs4,rw srv:/srv/share` → monté dans /mnt/auto/share au premier accès
- `* -fstype=nfs4 srv:/home/&` → joker : /mnt/auto/john monte srv:/home/john (& = la clé demandée)
- Disque local : `backup -fstype=ext4 :/dev/sdb1` (le : devant le périphérique est obligatoire)
- Montage direct (chemin absolu) : dans auto.master `/- /etc/auto.direct`, et dans la map `/data/nfs -fstype=nfs4 srv:/srv/share`
- `sudo systemctl restart autofs` → à relancer après toute modification des maps
- Tester : `ls /mnt/auto/share` (le dossier /mnt/auto/share n'existe pas tant qu'on n'y accède pas ; `ls /mnt/auto` peut sembler vide), puis `findmnt` pour voir le montage actif
- `sudo automount -f -v` → lance autofs au premier plan avec messages détaillés (arrêter le service avant) : pratique pour déboguer une map

## Alternative sans autofs : option fstab systemd

- `srv:/srv/share /mnt/nfs nfs4 noauto,x-systemd.automount,x-systemd.idle-timeout=60 0 0` → ligne fstab : monte au premier accès, démonte après 60 s d'inactivité
- Après modification : recharger systemd (`daemon-reload`, voir [systemd](systemd.md)), puis démarrer l'unité automount : `sudo systemctl start mnt-nfs.automount` (nom = chemin du point de montage avec des tirets)
- `systemctl list-units --type=automount` → liste les points de montage à la demande
