# GRUB et récupération

[← Index](../README.md)

## Configurer GRUB

- `/etc/default/grub` → réglages : `GRUB_TIMEOUT` (secondes d'attente du menu), `GRUB_DEFAULT` (entrée par défaut), `GRUB_CMDLINE_LINUX_DEFAULT` (paramètres noyau ajoutés au démarrage normal)
- `sudo update-grub` → régénère grub.cfg après modification (Debian / Ubuntu). Ne JAMAIS éditer grub.cfg à la main : il est écrasé
- `sudo grub2-mkconfig -o /boot/grub2/grub.cfg` → équivalent RHEL / Fedora (en UEFI, chemin sous /boot/efi/EFI/ selon la version)
- `sudo grub-install /dev/sda` → réinstalle GRUB sur le disque (RHEL : `grub2-install`)
- `sudo grub-set-default 1` → choisit l'entrée par défaut (avec `GRUB_DEFAULT=saved`) ; `sudo grub-reboot 1` → entrée pour le PROCHAIN démarrage seulement

## Modifier le démarrage « à chaud » (menu GRUB)

Afficher le menu : maintenir Shift (BIOS) ou appuyer sur Échap (UEFI) au démarrage. Sélectionner l'entrée, appuyer sur `e` pour l'éditer (temporaire), puis Ctrl+X ou F10 pour démarrer. Paramètres à ajouter à la fin de la ligne `linux` :

- `systemd.unit=rescue.target` → démarre en mode rescue (voir plus bas)
- `systemd.unit=emergency.target` → démarre en mode urgence
- `single` (ou `1`) → ancien nom du mode mono-utilisateur
- `init=/bin/bash` → lance directement un shell root (récupération de mot de passe sous Debian / Ubuntu)
- `rd.break` → s'arrête dans l'initramfs avant de monter la racine (récupération de mot de passe sous RHEL)

## Modes de secours systemd

- `sudo systemctl rescue` → bascule en mode rescue : mono-utilisateur, systèmes de fichiers montés, services de base
- `sudo systemctl emergency` → mode urgence : shell minimal, racine en lecture seule seulement
- En mode urgence, la racine est en lecture seule : la remonter en lecture-écriture (`remount,rw`, voir [Storage](storage.md)) avant de corriger un fichier
- `systemctl default` → revient à la cible de démarrage normale une fois la réparation faite

Changer de cible sans redémarrer : `isolate` (voir [Boot, processus](boot-processus.md)).

## Mot de passe root / utilisateur perdu

Debian / Ubuntu (ajouter `rw init=/bin/bash` à la ligne `linux`, en retirant `ro` s'il y est) :

```bash
mount -o remount,rw /
passwd john
sync
reboot -f
```

RHEL / Fedora (ajouter `rd.break` à la ligne `linux`) :

```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
touch /.autorelabel   # SELinux : ré-étiquette tout au prochain démarrage
exit
exit
```

## Système qui ne démarre plus

- Erreur dans fstab → arrivée en mode urgence : remonter la racine en lecture-écriture, corriger /etc/fstab (commenter la ligne fautive ou ajouter l'option `nofail`), puis tester avec `mount -a` (voir [Storage](storage.md))
- Système de fichiers corrompu → réparer avec `fsck` / `xfs_repair` depuis le mode rescue ou un live USB (voir [Storage](storage.md))
- Comprendre pourquoi : logs du démarrage (`journalctl -b`, voir [Logs](logs.md)) et messages noyau (`dmesg`)
- Démarrage lent : `systemd-analyze blame` (voir [systemd](systemd.md))

## Réparer GRUB depuis un live USB

Démarrer sur le live USB, identifier la partition racine avec `lsblk`, puis :

```bash
sudo mount /dev/sda2 /mnt
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt
grub-install /dev/sda
update-grub
exit
```

(en UEFI, monter aussi la partition EFI sur /mnt/boot/efi avant le chroot)

## initramfs

Mini-système chargé avant la racine : à régénérer quand on change mdadm.conf, crypttab, les modules du noyau ou LVM.

- `sudo update-initramfs -u` → régénère l'initramfs du noyau actuel (Debian / Ubuntu) ; `-k all` pour tous les noyaux
- `sudo dracut -f` → équivalent RHEL / Fedora
- `lsinitramfs /boot/initrd.img-$(uname -r) | head` → liste le contenu (Debian / Ubuntu ; RHEL : `lsinitrd`)
