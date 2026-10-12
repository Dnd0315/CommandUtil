# Storage

[← Index](../README.md)

## Disques et partitions

- `lsblk` → disques et partitions en arbre, avec les points de montage
- `sudo blkid /dev/sdb1` → UUID et type de système de fichiers (l'UUID sert dans fstab)
- `sudo fdisk -l` → liste tous les disques (taille, partitions)
- `findmnt -t xfs,ext4` → systèmes de fichiers montés, filtrés par type
- `sudo cfdisk /dev/sdb` → partitionnement en mode menu (plus simple) ; `fdisk` = mode commandes
- `lsblk -f` → idem avec le type de système de fichiers, le label et l'UUID
- `sudo fdisk /dev/sdb` → partitionnement interactif : `n` nouvelle partition, `d` supprime, `p` affiche, `t` change le type, `g` nouvelle table GPT (`o` = table DOS / MBR), `w` écrit et quitte, `q` quitte SANS rien écrire (rien n'est modifié tant qu'on n'a pas tapé `w`)
- `sudo parted /dev/sdb print` → table de partitions (parted gère GPT et les disques de plus de 2 To)
- `sudo parted -s /dev/sdb mklabel gpt mkpart primary ext4 1MiB 5GiB` → crée une table GPT et une partition d'environ 5 Go sans interaction (-s = script) ; ⚠️ `mklabel` efface la table existante
- `sudo partprobe /dev/sdb` → demande au noyau de relire la table de partitions sans redémarrer
- `sudo wipefs -a /dev/sdb1` → efface les signatures (système de fichiers, RAID, LVM, LUKS) pour réutiliser une partition (⚠️ irréversible)

## Swap sur partition

- `sudo mkswap /dev/sdb3` → formate la partition en swap
- `sudo swapon --verbose /dev/sdb3` → active ce swap
- `swapon --show` → liste les swaps actifs
- `sudo swapoff /dev/sdb3` → désactive ce swap

## Swap sur fichier

- `sudo dd if=/dev/zero of=/swapfile bs=1M count=128` → crée un fichier de 128 Mo rempli de zéros (bs = taille d'un bloc × count = nombre de blocs)
- `sudo fallocate -l 1G /swapfile` → crée un fichier de 1 Go (plus rapide)
- `sudo chmod 600 /swapfile` → seul root peut le lire (obligatoire)
- `sudo mkswap /swapfile` puis `sudo swapon /swapfile` → le formate et l'active
- fstab : `/swapfile none swap defaults 0 0` → swap actif au démarrage

## Systèmes de fichiers

- `sudo mkfs.ext4 /dev/sdb1` / `sudo mkfs.xfs /dev/sdb1` → crée un système de fichiers (efface les données) ; -L label pour nommer
- `sudo tune2fs -L data /dev/sdb1` → modifie un FS ext4 existant (label, options) ; `tune2fs -l` = afficher ses infos
- `sudo xfs_admin -L data /dev/sdb1` → équivalent pour XFS (démonté)
- `man mkfs.ext4` → toutes les options disponibles

## Montage

- `sudo mount /dev/sdb1 /mnt` → monte la partition sur /mnt (temporaire)
- `sudo umount /mnt` → démonte
- `/etc/fstab` → montages automatiques au démarrage. Colonnes : périphérique, point de montage, type, options, dump (0), ordre fsck (0 aucun, 1 racine, 2 autres). Exemple : `UUID=xxxx /data ext4 defaults 0 2`
- Après modification de fstab : recharger systemd (`daemon-reload`, voir [systemd](systemd.md)) puis `sudo mount -a` → monte tout fstab ; une erreur ici = machine qui ne démarre plus, donc TOUJOURS tester
- Options fstab utiles (4e colonne) : `defaults` (rw, suid, dev, exec, auto, async) / `ro` / `noexec` / `nosuid` / `noauto` (pas monté au démarrage) / `nofail` (ne bloque pas le démarrage si le disque est absent) / `_netdev` (attendre le réseau : NFS, iSCSI) / `usrquota,grpquota` (voir [Quotas](quotas.md))
- `sudo mount -o remount,rw /` → remonte un système de fichiers déjà monté avec de nouvelles options, sans le démonter (ici la racine en lecture-écriture ; `remount,ro` pour la lecture seule)
- `sudo mount -o loop disque.img /mnt` → monte un fichier image (ISO, image disque) comme un disque ; `losetup -a` → liste les périphériques loop actifs

## Réparer un système de fichiers

Le système de fichiers doit être DÉMONTÉ (pour la racine : mode rescue ou live USB, voir [GRUB et récupération](grub-recuperation.md)).

- `sudo fsck /dev/sdb1` → vérifie et répare (détecte le type tout seul) ; `fsck -y` répond oui à toutes les questions ; `fsck -n` = lecture seule, sans rien corriger
- `sudo e2fsck -f /dev/sdb1` → ext2 / ext3 / ext4 : force la vérification même si le système de fichiers semble propre
- `sudo xfs_repair /dev/sdb1` → XFS (`fsck` ne répare pas le XFS) ; `xfs_repair -n` = simulation

## Agrandir / réduire un système de fichiers

- `sudo resize2fs /dev/sdb1` → ext4 : adapte le système de fichiers à la taille de la partition ou du volume (agrandir est possible monté)
- `sudo resize2fs /dev/sdb1 2G` → réduit à 2 Go : démonter et lancer `e2fsck -f` avant, puis réduire la partition APRÈS
- `sudo xfs_growfs /data` → XFS : agrandit (on donne le POINT DE MONTAGE, système monté). XFS ne peut PAS être réduit
- Avec LVM, `lvresize --resizefs` fait tout (voir [LVM](lvm.md))

## Voir aussi

- [RAID logiciel (mdadm)](raid.md), [Chiffrement de disque (LUKS)](luks.md), [Quotas disque](quotas.md), [Montage à la demande (autofs)](autofs.md), [LVM](lvm.md)
