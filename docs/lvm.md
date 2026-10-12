# LVM (volumes logiques)

[← Index](../README.md)

Principe : disques → PV (volumes physiques) → regroupés en VG (groupe de volumes) → découpé en LV (volumes logiques), qu'on formate et monte comme une partition. PE = petit bloc d'espace élémentaire.

## Volumes physiques (PV)

- `sudo pvcreate /dev/sdc /dev/sdd` → prépare les disques pour LVM
- `sudo pvremove /dev/sdc` → retire le marquage LVM d'un disque
- `sudo pvmove /dev/sdc` → déplace toutes les données de ce PV vers les autres PV du groupe (à faire AVANT `vgreduce` pour retirer un disque en service)
- `sudo pvresize /dev/sdc` → prend en compte un disque agrandi (ex : disque virtuel étendu)
- `sudo lvmdiskscan` → liste tous les disques et ceux qui sont déjà des PV ; ne jamais toucher à ubuntu-vg (système)

## Groupes de volumes (VG)

- `sudo vgcreate vg1 /dev/sdc /dev/sdd` → crée le groupe vg1 avec ces deux disques
- `sudo vgextend vg1 /dev/sde` → ajoute un disque au groupe / `vgreduce` → en retire un
- `sudo vgremove vg1` → supprime le groupe (après avoir supprimé ses LV)
- `sudo vgchange -ay vg1` → active tous les volumes du groupe (`-an` les désactive)
- `sudo vgrename vg1 vg_data` → renomme le groupe (les chemins /dev/vg1/… et les lignes fstab changent)

## Volumes logiques (LV)

- `sudo lvcreate --size 2G --name lv1 vg1` → crée un volume de 2 Go nommé lv1 dans vg1 (`--extents 100%FREE` = tout l'espace libre)
- `sudo mkfs.ext4 /dev/vg1/lv1` → formate le volume (puis mount / fstab, voir [Storage](storage.md)) ; /dev/mapper/vg1-lv1 désigne le même volume
- `sudo lvresize --size +1G --resizefs /dev/vg1/lv1` → agrandit de 1 Go ET redimensionne le système de fichiers (sinon l'espace n'est pas utilisable) ; `lvextend -r` = équivalent (ex : `lvextend -l +100%FREE -r /dev/vg1/lv1` → prend tout l'espace libre du groupe). XFS ne peut pas être réduit
- `sudo lvreduce --size -1G --resizefs /dev/vg1/lv1` → réduit de 1 Go (ext4 seulement ; le système de fichiers est démonté pendant l'opération ; sauvegarder avant). Ne JAMAIS réduire sans `--resizefs` : perte de données
- `sudo lvrename vg1 lv1 lv_data` → renomme le volume
- `sudo lvremove /dev/vg1/lv1` → supprime le volume (démonter avant ; les données sont perdues)

## Snapshots

- `sudo lvcreate --size 1G --snapshot --name snap1 /dev/vg1/lv1` → instantané de lv1 (la taille = la place pour les CHANGEMENTS faits tant que le snapshot existe ; plein = snapshot invalide)
- `sudo mount -o ro /dev/vg1/snap1 /mnt/snap` → lit le snapshot (sauvegarde, comparaison) ; en XFS, ajouter l'option `nouuid`
- `sudo lvconvert --merge /dev/vg1/snap1` → ramène lv1 à l'état du snapshot (démonter lv1 avant, sinon la fusion se fait à la prochaine activation) ; le snapshot disparaît ensuite
- Supprimer un snapshot devenu inutile : `lvremove`, comme un volume

## Consulter

- `pvs` / `vgs` / `lvs` → résumé des PV / VG / LV (pvdisplay etc. pour le détail)
- `sudo lvs -a -o +devices` → ajoute la colonne des disques qui portent chaque volume
