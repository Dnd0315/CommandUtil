# Liste des commandes utiles pour ingénieur IT

Aide-mémoire (LFCS) : un fichier par section dans [`docs/`](docs/). Chaque commande n'apparaît qu'une seule fois ; les renvois entre sections sont des liens.

Format : `commande` → ce qu'elle fait / quand l'utiliser

## Comptes et sécurité

- [Utilisateurs](docs/utilisateurs.md) — useradd, usermod, chage, groupes, sudo, su, connexions, valeurs par défaut (skel, login.defs), ulimit
- [PAM](docs/pam.md) — /etc/pam.d, pam_limits, pwquality, faillock, pam_wheel, mkhomedir
- [LDAP / NSS](docs/ldap-nss.md) — comptes centralisés, nsswitch.conf, nslcd, getent
- [Permissions](docs/permissions.md) — chown, chmod, SUID/SGID/sticky, ACL (dont par défaut), umask, chattr, liens, stat
- [SSH](docs/ssh.md) — clés, client et config, scp / sftp, tunnels, sshd_config
- [SSL / TLS](docs/ssl-tls.md) — paires de clés, CSR, certificats auto-signés (openssl)
- [SELinux / AppArmor](docs/selinux-apparmor.md) — modes, étiquettes, ports, booléens, audit2allow, profils AppArmor

## Fichiers et texte

- [Commandes de base](docs/commandes-de-base.md) — ls, cp, mv, rm, less, locate, man / info, historique et raccourcis
- [find](docs/find.md) — critères, actions (-exec), combiner avec ET / OU / NON
- [Créer / écrire dans un fichier](docs/fichiers-et-redirections.md) — echo, tee, heredoc, redirections stdout / stderr
- [Texte](docs/texte.md) — sed, cut, awk, sort, uniq, diff, grep, xargs, tr, paste, empreintes
- [Regex](docs/regex.md) — regex de base et étendues
- [Archives](docs/archives.md) — tar, gzip, bzip2, xz, zip, rsync
- [Scripts bash](docs/scripting-bash.md) — variables, arguments, tests, boucles, case, fonctions, exemple
- [Git](docs/git.md) — clone, branches, merge, dossier vide

## Système et services

- [Variables d'environnement](docs/variables-environnement.md) — export, PATH, alias, fichiers de profil (/etc/profile.d, .bashrc)
- [Boot, processus](docs/boot-processus.md) — reboot, shutdown, cibles, nice, ps, top, kill, signaux, strace
- [GRUB et récupération](docs/grub-recuperation.md) — GRUB, modes rescue / emergency, mot de passe perdu, initramfs
- [Modules du noyau](docs/modules-noyau.md) — lsmod, modprobe, modules-load.d, modprobe.d
- [systemd (services)](docs/systemd.md) — systemctl, diagnostic, override, créer un service
- [Logs](docs/logs.md) — /var/log, dmesg, journalctl, rsyslog, logrotate
- [cron / at](docs/cron-at.md) — cron, dossiers cron.*, at, timers systemd
- [Paquets (apt / dpkg)](docs/paquets.md) — install, hold, dpkg, dépôts tiers
- [Système](docs/systeme.md) — df, du, free, uname, hostnamectl, sysctl
- [Time](docs/time.md) — timedatectl, heure manuelle, timesyncd, chrony
- [Monitor](docs/monitor.md) — iostat, pidstat, vmstat

## Réseau

- [Réseau](docs/reseau.md) — ip, IPv6, IP statique, routage, netplan, bond / bridge, DNS, ss, ufw, iptables (NAT, chaînes, persistance)
- [Dépannage réseau](docs/depannage-reseau.md) — méthode, ping, traceroute, dig, nc, curl, tcpdump, ethtool
- [nftables](docs/nftables.md) — tables, chaînes, règles, NAT, persistance
- [Serveur DNS (BIND9)](docs/dns-bind.md) — serveur cache et redirecteurs, zones directe et inverse, serveur secondaire, rndc, dnsmasq
- [Apache (serveur web)](docs/apache.md) — hôtes virtuels, a2ensite / a2enmod, accès restreint (IP, htpasswd), HTTPS, logs, reverse proxy
- [Nginx (reverse proxy)](docs/nginx.md) — sites-available / sites-enabled, nginx -t

## Stockage

- [Storage](docs/storage.md) — lsblk, fdisk / parted, swap, mkfs, fsck, resize, mount, fstab
- [LVM](docs/lvm.md) — PV, VG, LV, redimensionnement, snapshots
- [RAID logiciel (mdadm)](docs/raid.md) — niveaux, création, panne et remplacement, persistance
- [Chiffrement de disque (LUKS)](docs/luks.md) — cryptsetup, crypttab, fichiers clés
- [Quotas disque](docs/quotas.md) — ext4 (quota) et XFS (xfs_quota)
- [Montage à la demande (autofs)](docs/autofs.md) — maps autofs, option fstab x-systemd.automount
- [SSHFS](docs/sshfs.md) — monter un dossier distant via SSH
- [NBD](docs/nbd.md) — disque réseau (nbd-client)
- [NFS](docs/nfs.md) — exports, exportfs, montage client, root_squash

## Virtualisation

- [VM / conteneurs](docs/vm-conteneurs.md) — virsh, virt-install, qemu-img, snapshots, docker (build, volumes, réseaux), podman

## Autres distributions

- [Équivalents RHEL / Fedora](docs/rhel-fedora.md) — dnf / rpm, firewalld, nmcli, noms de paquets et de services

## Examen

- [À ne pas oublier le jour J](docs/jour-j.md) — machine de travail, persistance, vérifications, aide
