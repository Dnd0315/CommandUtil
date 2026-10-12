# Dépannage réseau

[← Index](../README.md)

## Méthode : du plus bas au plus haut

1. Interface active ? → `ip link` (voir [Réseau](reseau.md))
2. Adresse IP correcte ? → `ip addr` (voir [Réseau](reseau.md))
3. Passerelle joignable ? → `ping` de la passerelle (`ip route` la donne)
4. Internet joignable ? → `ping` d'une IP publique (ex : 8.8.8.8)
5. DNS OK ? → `ping` d'un NOM, ou `dig` : si l'IP répond mais pas le nom, c'est le DNS
6. Service joignable ? → `nc -vz`, `curl`, `ss` côté serveur

## Connectivité

- `ping -c 4 8.8.8.8` → 4 paquets ICMP (sans -c, ping tourne jusqu'à Ctrl+C) ; `ping -6 ::1` → en IPv6
- `traceroute 8.8.8.8` → chaque routeur traversé (paquet traceroute à installer) ; `tracepath 8.8.8.8` → équivalent, installé par défaut
- `mtr -rwc 10 8.8.8.8` → ping + traceroute combinés, en rapport (-r) sur 10 envois (-c)
- `nc -vz serveur 22` → teste si un port TCP est joignable (-z = sans envoyer de données, -v = verbeux) ; `nc -vzu serveur 53` pour UDP
- `nc -l 9000` → écoute sur le port 9000 (test) puis `nc serveur 9000` depuis une autre machine pour y écrire ; selon la version de netcat : `nc -l -p 9000`
- `sudo fuser -v 80/tcp` → quel processus utilise le port 80

## DNS

- `dig example.com` → requête DNS détaillée (paquet dnsutils ; RHEL : bind-utils)
- `dig +short example.com` → seulement la réponse ; `dig example.com MX` → un type d'enregistrement précis (A, AAAA, MX, NS, TXT…)
- `dig @8.8.8.8 example.com` → interroge CE serveur DNS au lieu de celui du système
- `dig -x 8.8.8.8` → résolution inverse (IP → nom)
- `nslookup example.com` / `host example.com` → interrogations plus simples
- `getent hosts example.com` → résolution comme le système la fait vraiment (/etc/hosts puis DNS, selon nsswitch.conf)
- `resolvectl query example.com` → résolution via systemd-resolved (indique le serveur et l'interface utilisés)

## HTTP

- `curl -v http://serveur` → détail de la connexion et des en-têtes (requête et réponse)
- `curl -s -o /dev/null -w "%{http_code}\n" http://serveur` → seulement le code HTTP (200, 403, 502…)
- `curl -o fichier.html http://serveur/page` → enregistre la réponse ; `-L` suit les redirections
- `wget -O fichier http://serveur/fichier` → télécharge un fichier

## Interfaces, voisins, trafic

- `ip neigh` → table des voisins (ARP / NDP) : quelles adresses MAC sont connues
- `ip -s link show eth0` → compteurs de paquets, erreurs et paquets perdus (drops)
- `sudo ethtool eth0` → vitesse, duplex et « Link detected: yes/no » (câble branché ?)
- `hostname -I` → toutes les adresses IP de la machine
- `networkctl status` → état des interfaces gérées par systemd-networkd (renderer networkd de netplan)
- `ss -tn state established` → connexions TCP établies ; `ss -s` → résumé des sockets

## tcpdump (capture de paquets)

- `sudo tcpdump -i eth0` → affiche le trafic de l'interface (Ctrl+C pour arrêter) ; `-i any` pour toutes
- `sudo tcpdump -i eth0 -nn port 80` → sans résolution de noms ni de ports (-nn), seulement le port 80
- `sudo tcpdump -i any -nn icmp -c 5` → 5 paquets ICMP (ping) puis arrêt
- `sudo tcpdump -i eth0 -nn host 10.0.0.5 and tcp port 22` → filtres combinés (host, port, net, src, dst, and, or, not)
- `sudo tcpdump -i eth0 -w capture.pcap` → enregistre dans un fichier ; `tcpdump -r capture.pcap` → le relit
