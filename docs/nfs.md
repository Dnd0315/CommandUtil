# NFS (partage réseau)

[← Index](../README.md)

## Serveur

- `sudo apt install nfs-kernel-server` → installation
- `/etc/exports` → `/srv/share 10.0.0.0/24(rw,sync,no_subtree_check)` = partage /srv/share en lecture-écriture pour ce réseau (pas d'espace avant la parenthèse)
- Partage en LECTURE SEULE pour un réseau : `/nfs/share 192.168.10.0/24(ro,sync,no_subtree_check)` dans /etc/exports
- `sudo exportfs -ra` → recharge /etc/exports après chaque modification
- `sudo exportfs -v` → vérifie ce qui est partagé et avec quelles options
- `showmount -e` → liste les partages exportés par cette machine (depuis un client : `showmount -e serveur`)
- `service nfs-kernel-server status` → vérifier que le service tourne

## Client

- Tester que le port NFS (2049) est joignable : `nc -vz` (voir [Dépannage réseau](depannage-reseau.md))
- `sudo mount srv:/srv/share /mnt/nfs` → monte le partage distant (créer d'abord le point de montage avec `mkdir -p`) ; test : `touch` → « Read-only file system » si ro
- fstab client : `srv:/srv/share /mnt/nfs nfs defaults 0 0`

## root_squash : « Permission denied » en écriture

⚠️ root_squash = activé PAR DÉFAUT (visible dans `exportfs -v`) : le root du CLIENT est traité comme l'utilisateur sans droits « nobody » côté serveur → « Permission denied » en écriture si le dossier partagé appartient à root.

- Solution A (la plus sûre) : `sudo chown nobody:nogroup /srv/share` → nobody peut écrire ; les fichiers créés depuis le client appartiennent à nobody
- Solution B : ajouter `no_root_squash` dans les options (`/srv/share 10.0.0.0/24(rw,sync,no_subtree_check,no_root_squash)`) puis `exportfs -ra` → le root du client reste root (moins sûr, seulement si la consigne le demande)
