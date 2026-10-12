# Quotas disque

[← Index](../README.md)

Quota = limite d'espace disque (blocs) et/ou de nombre de fichiers (inodes) par utilisateur ou par groupe, sur un système de fichiers.

- soft limit = limite « souple » : peut être dépassée pendant le délai de grâce (grace period)
- hard limit = plafond absolu, jamais dépassable
- Les blocs sont comptés en Ko (blocs de 1 Ko) dans edquota et setquota

## ext4

- `sudo apt install quota` → installation
- Ajouter les options dans fstab : `UUID=xxxx /data ext4 defaults,usrquota,grpquota 0 2` (usrquota = quotas utilisateurs, grpquota = quotas groupes), puis `sudo mount -o remount /data`
- `sudo quotacheck -cugm /data` → crée les fichiers de quotas aquota.user et aquota.group à la racine du système de fichiers (-c créer, -u utilisateurs, -g groupes, -m sans remonter en lecture seule)
- `sudo tune2fs -O quota /dev/sdb1` → alternative sur ext4 récent (système de fichiers démonté) : quotas gérés en interne, plus besoin de quotacheck ; garder usrquota / grpquota dans fstab
- `sudo quotaon -v /data` → active les quotas / `sudo quotaoff -v /data` → les désactive. Sur un noyau « cloud » Ubuntu, si quotaon échoue, installer `linux-modules-extra-$(uname -r)` (module quota_v2)

## Définir les limites

- `sudo edquota -u john` → ouvre un éditeur : colonnes blocks / soft / hard et inodes / soft / hard pour john
- `sudo edquota -g dev` → idem pour le groupe dev
- `sudo setquota -u john 100000 120000 0 0 /data` → non interactif : soft blocs, hard blocs, soft inodes, hard inodes (0 = pas de limite)
- `sudo edquota -p john jane` → copie les quotas de john vers jane
- `sudo edquota -t` → délai de grâce (blocs et inodes)

## Consulter

- `sudo quota -u john` → quotas et consommation de john (`quota -vs` pour l'utilisateur courant, en unités lisibles)
- `sudo repquota -as` → rapport de tous les systèmes de fichiers avec quotas (-a tous, -s unités lisibles)
- Tester : se connecter en tant que l'utilisateur et créer un gros fichier → « Disk quota exceeded » une fois la hard limit atteinte

## XFS

- Les quotas XFS s'activent au MONTAGE (pas de remount possible pour les activer) : `UUID=xxxx /data xfs defaults,usrquota,grpquota 0 0`, puis démonter / remonter
- `sudo xfs_quota -x -c 'limit bsoft=100m bhard=120m john' /data` → limites de blocs pour john (isoft / ihard pour les inodes)
- `sudo xfs_quota -x -c 'report -h' /data` → rapport lisible des quotas
