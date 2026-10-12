# Équivalents RHEL / Rocky / Alma / Fedora

[← Index](../README.md)

Le reste de cet aide-mémoire suit Debian / Ubuntu (apt, netplan, ufw, AppArmor). L'examen étant indépendant de la distribution, voici les équivalents RHEL. Tout le reste (systemd, journalctl, ip, ss, LVM, mdadm, SSH, find, sed…) est identique.

## Paquets : dnf / rpm

(`yum` = alias de `dnf` sur RHEL 8 et plus)

- `sudo dnf install nginx` → installe (équivalent de `apt install`)
- `sudo dnf remove nginx` → désinstalle (équivalent de `apt remove` + `autoremove`)
- `sudo dnf upgrade` → met à jour tous les paquets (les métadonnées sont rafraîchies automatiquement ; équivalent de `apt update` + `apt upgrade`)
- `dnf search nginx` / `dnf info nginx` → cherche / détails
- `dnf provides /usr/bin/ls` → quel paquet fournit ce fichier (équivalent de `dpkg -S` ; marche aussi pour un fichier non installé : `dnf provides '*/semanage'`)
- `dnf list installed` → paquets installés ; `dnf history` → historique des transactions
- `dnf repolist` → dépôts actifs ; `/etc/yum.repos.d/*.repo` → fichiers de dépôts
- `sudo dnf config-manager --add-repo https://repo.example.com/x.repo` → ajoute un dépôt (paquet dnf-plugins-core)
- `sudo dnf install ./paquet.rpm` → installe un .rpm local en résolvant les dépendances
- `rpm -qa` → liste tous les paquets installés ; `rpm -qa | grep nginx`
- `rpm -qi nginx` → informations sur un paquet installé
- `rpm -ql nginx` → fichiers installés par le paquet (équivalent de `dpkg --listfiles`) ; `rpm -qc nginx` → seulement ses fichiers de configuration
- `rpm -qf /usr/bin/ls` → quel paquet installé possède ce fichier
- `rpm -V nginx` → vérifie si des fichiers du paquet ont été modifiés

Noms de paquets / services différents :

- Apache : `httpd` (Debian : apache2) / SSH : service `sshd` (Debian : ssh) / cron : paquet `cronie`, service `crond` (Debian : cron)
- NFS serveur : `nfs-utils`, service `nfs-server` (Debian : nfs-kernel-server) / NTP : `chrony`, service `chronyd` (Debian : chrony)
- Serveur DNS : paquet `bind`, service `named`, config dans `/etc/named.conf` et zones dans `/var/named/` (Debian : bind9, `/etc/bind/`) ; pare-feu : `sudo firewall-cmd --permanent --add-service=dns` (voir [Serveur DNS](dns-bind.md))
- `dig` : paquet `bind-utils` (Debian : dnsutils) / `semanage` : paquet `policycoreutils-python-utils`

## Pare-feu : firewalld

Zones (public par défaut) ; sans `--permanent`, la règle est perdue au rechargement / redémarrage.

- `sudo systemctl enable --now firewalld` (voir [systemd](systemd.md)) puis `sudo firewall-cmd --state` → état du pare-feu
- `sudo firewall-cmd --get-active-zones` → zones actives et leurs interfaces
- `sudo firewall-cmd --list-all` → règles de la zone par défaut (services, ports, redirections…)
- `sudo firewall-cmd --permanent --add-service=http` → autorise un service (équivalent de `ufw allow`)
- `sudo firewall-cmd --permanent --add-port=8080/tcp` → autorise un port
- `sudo firewall-cmd --permanent --remove-service=http` → retire la règle
- `sudo firewall-cmd --reload` → applique les règles permanentes
- `sudo firewall-cmd --runtime-to-permanent` → rend permanentes les règles ajoutées sans `--permanent`
- `sudo firewall-cmd --permanent --zone=internal --add-source=10.0.0.0/24` → classe ce réseau dans la zone internal
- `sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.0/24" port port="80" protocol="tcp" accept'` → règle fine : port 80 seulement depuis ce réseau
- `sudo firewall-cmd --permanent --add-forward-port=port=8080:proto=tcp:toport=80` → redirige le port 8080 vers le port local 80 (ajouter `:toaddr=IP` pour une autre machine)
- `sudo firewall-cmd --permanent --add-masquerade` → NAT / masquerade

## Réseau : NetworkManager (nmcli)

Remplace netplan. Une « connexion » (profil) est appliquée à un périphérique.

- `nmcli device status` → périphériques et leur état ; `nmcli connection show` → profils
- `sudo nmcli connection add type ethernet con-name lan ifname eth1 ipv4.method manual ipv4.addresses 10.0.0.40/24 ipv4.gateway 10.0.0.1 ipv4.dns 8.8.8.8` → IP statique (`ipv4.method auto` = DHCP)
- `sudo nmcli connection modify lan +ipv4.addresses 10.0.0.41/24` → ajoute une adresse (sans le +, remplace) ; `ipv4.dns`, `ipv4.gateway`, `ipv4.routes "192.168.5.0/24 10.0.0.1"` se modifient de la même façon
- `sudo nmcli connection up lan` → applique / active (nécessaire après un modify) ; `down` désactive
- `sudo nmcli connection delete lan` → supprime le profil
- `sudo nmcli connection add type bond con-name bond0 ifname bond0 bond.options "mode=active-backup"` puis `sudo nmcli connection add type ethernet slave-type bond con-name bond0-p1 ifname eth1 master bond0` → bond (répéter pour chaque carte)
- `sudo nmcli connection add type bridge con-name br0 ifname br0` puis `sudo nmcli connection add type bridge-slave con-name br0-p1 ifname eth1 master br0` → bridge
- `sudo nmcli general hostname srv1` → change le nom de la machine
- `nmtui` → même chose en mode menu ; profils dans `/etc/NetworkManager/system-connections/`

## Autres différences

- Groupe administrateur : `wheel` (au lieu de `sudo`) → `sudo usermod -aG wheel john`
- SELinux actif (enforcing) par défaut, pas d'AppArmor : voir [SELinux / AppArmor](selinux-apparmor.md)
- GRUB : `grub2-mkconfig`, `dracut` : voir [GRUB et récupération](grub-recuperation.md)
- Logs : `/var/log/messages` (au lieu de syslog), `/var/log/secure` (au lieu de auth.log), `/var/log/cron`
- Configuration des services : `/etc/sysconfig/` (au lieu de `/etc/default/`)
