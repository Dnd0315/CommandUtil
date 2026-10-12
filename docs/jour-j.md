# À ne pas oublier le jour J

[← Index](../README.md)

⚠️ Toujours taper soi-même les guillemets ' et les doubles tirets -- (Notes les transforme en ’ et —).

## Où travaille-t-on ?

Vérifier sur QUELLE machine on travaille (prompt user@HOST) : si la tâche dit node01 → `ssh node01` d'abord, puis `exit` pour revenir à la machine principale une fois la tâche finie.

## Persistance

Ce qui doit survivre au redémarrage :

- fstab → [Storage](storage.md)
- systemctl enable → [systemd](systemd.md)
- netplan → [Réseau](reseau.md)
- sysctl.d → [Système](systeme.md)
- limits.conf → [Utilisateurs](utilisateurs.md)
- setsebool -P → [SELinux / AppArmor](selinux-apparmor.md)
- iptables-persistent → [Réseau](reseau.md)

## Appliquer la configuration

Après avoir modifié la config d'un service : redémarrer le service (nslcd, systemd-timesyncd, ssh…), sinon le changement n'est pas actif. Voir [systemd](systemd.md).

## Vérifier chaque tâche

- `mount -a` → [Storage](storage.md)
- `systemctl status` → [systemd](systemd.md)
- `ip a` → [Réseau](reseau.md)
- `id user` → [Utilisateurs](utilisateurs.md)
- `getent passwd` → [LDAP / NSS](ldap-nss.md)

## Trouver de l'aide

Voir [Commandes de base](commandes-de-base.md) : `man`, `man -k`, `--help`, `info`.
