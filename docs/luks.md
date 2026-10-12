# Chiffrement de disque (LUKS)

[← Index](../README.md)

LUKS = chiffrement d'une partition ou d'un volume entier (dm-crypt). Le périphérique chiffré (/dev/sdb1) est « ouvert » sous un nom, ce qui crée /dev/mapper/NOM ; c'est ce périphérique mapper qu'on formate et qu'on monte. Fonctionne aussi sur un volume LVM (/dev/vg1/lv1).

## Créer et utiliser

- `sudo apt install cryptsetup` → installation
- `sudo cryptsetup luksFormat /dev/sdb1` → chiffre la partition (⚠️ EFFACE les données ; taper YES en majuscules, puis choisir la passphrase)
- `sudo cryptsetup luksOpen /dev/sdb1 secret` → ouvre le volume chiffré (demande la passphrase) et crée /dev/mapper/secret ; `open` est un alias
- Formater /dev/mapper/secret et le monter comme une partition ordinaire (`mkfs`, `mount` : voir [Storage](storage.md))
- `sudo cryptsetup luksClose secret` → referme le volume (démonter AVANT) ; `close` est un alias
- `sudo cryptsetup status secret` → état du volume ouvert (chiffrement, taille, périphérique)
- `sudo cryptsetup luksDump /dev/sdb1` → en-tête LUKS : version, algorithme, emplacements de clés (keyslots) utilisés
- `sudo cryptsetup luksUUID /dev/sdb1` → UUID du volume chiffré (celui à mettre dans crypttab)

## Gérer les clés

- `sudo cryptsetup luksAddKey /dev/sdb1` → ajoute une 2e passphrase (jusqu'à 8 emplacements) ; ajouter un fichier clé : `sudo cryptsetup luksAddKey /dev/sdb1 /root/luks.key`
- `sudo cryptsetup luksRemoveKey /dev/sdb1` → supprime une passphrase (celle à retirer est demandée)
- `sudo cryptsetup luksChangeKey /dev/sdb1` → change une passphrase
- Fichier clé : `sudo dd if=/dev/urandom of=/root/luks.key bs=1024 count=4` puis chmod 600 (lisible par root seulement)
- `sudo cryptsetup luksHeaderBackup /dev/sdb1 --header-backup-file hdr.img` → sauvegarde l'en-tête (en-tête perdu = données perdues)

## Ouverture automatique au démarrage

- `/etc/crypttab` → un volume par ligne : `nom  périphérique  clé  options`. Exemple avec passphrase demandée au boot : `secret UUID=xxxx-xxxx none luks` ; avec fichier clé : `secret UUID=xxxx-xxxx /root/luks.key luks`
- L'UUID de crypttab est celui du périphérique CHIFFRÉ (`luksUUID`), pas celui de /dev/mapper/secret
- `/etc/fstab` → monter le périphérique mapper : `/dev/mapper/secret /data ext4 defaults 0 2`
- `sudo cryptdisks_start secret` → ouvre le volume décrit dans crypttab sans redémarrer (Debian / Ubuntu) ; puis tester le montage avec `mount -a`
- Après modification de crypttab : régénérer l'initramfs si le volume est nécessaire au boot (voir [GRUB et récupération](grub-recuperation.md))
