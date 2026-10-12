# Réseau

[← Index](../README.md)

Voir aussi : [Dépannage réseau](depannage-reseau.md), [nftables](nftables.md), [Équivalents RHEL (firewalld, nmcli)](rhel-fedora.md).

## Ports en écoute

- `sudo ss -ltunp` → ports en écoute : l listening, t TCP, u UDP, n numérique, p processus (netstat = ancienne version)
- `sudo lsof -i -P -n | grep LISTEN | grep 8080` → vérifie si un service écoute sur un port en particulier (ici 8080)

## Interfaces et adresses IP

Commandes ip = changements TEMPORAIRES (perdus au redémarrage) ; persistant = netplan.

- `ip link` → liste les interfaces réseau et leur état (UP/DOWN)
- `ip addr` (ou `ip a`) → interfaces avec leurs adresses IP ; `ip -4 addr show eth0` → seulement l'IPv4 de eth0
- `sudo ip link set dev enp0s8 up` → active l'interface (down = désactive)
- `sudo ip addr add 10.0.0.40/24 dev enp0s8` → ajoute une IP à l'interface
- `sudo ip addr delete 10.0.0.40/24 dev enp0s8` → retire cette IP

## IPv6

- `ip -6 addr show eth0` → seulement l'IPv6 de eth0 (`::1` = localhost ; `fe80::` = adresse locale au lien, créée automatiquement ; une adresse globale commence par 2 ou 3)
- `sudo ip -6 addr add 2001:db8::10/64 dev eth0` → ajoute une adresse IPv6 (temporaire)
- `ip -6 route` → table de routage IPv6
- `sudo ip -6 route add default via 2001:db8::1 dev eth0` → passerelle IPv6 par défaut (avec une passerelle de lien local comme `fe80::1`, le `dev` est obligatoire)
- `ip -6 neigh` → voisins IPv6 (équivalent de l'ARP) ; test de connectivité : `ping -6` (voir [Dépannage réseau](depannage-reseau.md))

## IP statique IPv4 / IPv6 (netplan)

Fichier `/etc/netplan/99-static.yaml`, puis `sudo netplan apply` :

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 10.0.0.40/24
        - 2001:db8::40/64
      routes:
        - to: default
          via: 10.0.0.1
        - to: default
          via: 2001:db8::1
      nameservers:
        addresses: [8.8.8.8, 2001:4860:4860::8888]
```

- `dhcp4: yes` (et `dhcp6: yes`) à la place de `addresses` pour obtenir l'adresse par DHCP
- Sur une image cloud, cloud-init peut régénérer sa propre config netplan : y mettre `network: {config: disabled}` dans `/etc/cloud/cloud.cfg.d/99-disable-network-config.cfg`

## Routage

Routage = la machine consulte sa table de routage pour savoir PAR OÙ faire sortir chaque paquet (« pour aller vers tel réseau, passe par telle porte »).

Dans une route : default = tout ce qui ne correspond à aucune autre ligne (ex : Internet) ; via = le routeur (passerelle) à qui confier le paquet ; dev = l'interface de sortie.

- `ip route` → affiche la table de routage
- `sudo ip route add default via 10.0.0.1` → définit la passerelle par défaut
- `sudo ip route add 192.168.5.0/24 via 10.0.0.1 dev enp0s8` → route vers un réseau précis via une passerelle
- `sudo ip route del 192.168.5.0/24` → supprime la route
- Routes persistantes : dans netplan, sous l'interface (une entrée `- to: … / via: …` par route) :

  ```yaml
  routes:
    - to: default
      via: 10.0.0.1
  ```

## Netplan (config persistante)

- `/etc/netplan/*.yaml` → configuration réseau persistante (indentation YAML stricte, espaces uniquement)
- `sudo netplan get` → affiche la config actuelle
- `sudo netplan try` → applique avec retour arrière automatique si tu ne confirmes pas (sécurité en SSH)
- `sudo netplan apply` → applique définitivement
- `sudo netplan generate` → valide la config et génère les fichiers du renderer sans l'appliquer (repère les erreurs de YAML) ; `sudo netplan --debug apply` → applique avec des messages détaillés
- Les fichiers netplan doivent être en chmod 600 (un netplan récent avertit « Permissions … are too open » sinon)

## Bond et Bridge

Bond (bond0) = plusieurs cartes physiques regroupées en UNE interface logique → redondance (si une carte tombe, l'autre prend le relais) ou plus de débit. Mode le plus courant : active-backup (une carte active, l'autre en secours).

Bridge (br0) = switch virtuel dans la machine : relie des interfaces entre elles. Usage typique : brancher la carte physique + les VM, qui apparaissent alors sur le réseau comme de vraies machines.

Pour les deux : l'IP (ou `dhcp4: yes`) va sur bond0 / br0, PAS sur les cartes membres (`dhcp4: no`).

⚠️ /usr/share/doc/netplan/examples/ peut ne PAS exister (absent au Mock 2) → connaître ces modèles par cœur.

BRIDGE : créer `/etc/netplan/99-bridge.yaml` puis `sudo netplan apply`

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth1:
      dhcp4: no
    eth2:
      dhcp4: no
  bridges:
    bridge1:
      dhcp4: yes
      interfaces: [eth1, eth2]
```

BOND : créer `/etc/netplan/99-bond.yaml` puis `sudo netplan apply`

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth1:
      dhcp4: no
    eth2:
      dhcp4: no
  bonds:
    bond0:
      dhcp4: yes
      interfaces: [eth1, eth2]
      parameters:
        mode: active-backup
```

- `ip addr show bridge1` / `bridge link` → vérifie le bridge
- `cat /proc/net/bonding/bond0` → vérifie le bond (mode, carte active) ; `ip addr show bond0` → IP obtenue
- `sudo ip link del br0` → supprime l'interface br0 (temporaire : retirer aussi le bloc du fichier netplan)

## DNS et résolution de noms

- `cat /etc/resolv.conf` → serveurs DNS utilisés
- `resolvectl status` → DNS par interface (systemd-resolved)
- `/etc/hosts` → résolution de noms locale, prioritaire sur le DNS
- `/etc/nsswitch.conf` → ligne `hosts: files dns` = ordre de résolution (d'abord /etc/hosts, ensuite le DNS)
- `/etc/hostname` → nom de la machine (changer le nom : voir [Système](systeme.md)) ; l'ajouter aussi à /etc/hosts en face de 127.0.1.1 évite des lenteurs avec sudo
- DNS persistant : `nameservers:` dans netplan (voir plus haut) ou `/etc/systemd/resolved.conf` → `DNS=8.8.8.8 1.1.1.1` et `Domains=exemple.lan`, puis `sudo systemctl restart systemd-resolved`
- /etc/resolv.conf est un lien géré par systemd-resolved (vers /run/systemd/resolve/stub-resolv.conf) : ne pas l'éditer à la main

## Pare-feu : ufw

- `sudo ufw enable` → active le pare-feu (autoriser SSH AVANT en connexion distante)
- `sudo ufw status numbered` → liste les règles numérotées
- `sudo ufw allow 22/tcp` → autorise le port 22 en TCP entrant
- `sudo ufw allow from 10.0.0.0/24 to any port 80` → autorise le port 80 seulement depuis ce réseau
- `sudo ufw deny out 25` → bloque le trafic SORTANT vers le port 25
- `sudo ufw delete 3` → supprime la règle n°3

## iptables : redirection de port / NAT

DNAT = notre machine reçoit un paquet et le RENVOIE vers une autre machine / un autre port. MASQUERADE = remplace l'adresse source par celle de notre machine pour que la réponse revienne par nous (notre machine sert de routeur entre les 2 réseaux).

- `sudo iptables -t nat -A PREROUTING -p tcp -s 10.5.5.0/24 --dport 81 -j DNAT --to-destination 192.168.5.2:80` → les paquets venant de 10.5.5.0/24 sur le port 81 sont redirigés vers 192.168.5.2:80 (-t nat = table NAT ; -A = ajouter ; PREROUTING = dès l'arrivée ; -p tcp obligatoire pour utiliser --dport ; -s = source)
- `sudo iptables -t nat -A POSTROUTING -s 10.5.5.0/24 -j MASQUERADE` → masquerade : les réponses repassent par notre machine
- `sudo iptables -t nat -L -n -v` → vérifie les règles NAT (-L = lister, -n = numérique)
- `sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 6000 -j REDIRECT --to-port 6001` → redirige le port 6000 vers le port LOCAL 6001 de la même machine (REDIRECT = vers cette machine ; DNAT = vers une autre machine). Vérifier : `curl data-002:6000` depuis une autre machine
- `sudo iptables -F -t nat` → efface les règles NAT
- Pour qu'une machine route vraiment le trafic : `net.ipv4.ip_forward=1` (voir [Système](systeme.md))

## iptables : chaînes, tables, actions

Les chaînes : INPUT, OUTPUT, FORWARD (+ PREROUTING, POSTROUTING). Une chaîne = une ÉTAPE du parcours d'un paquet dans la machine. La chaîne dépend de la DESTINATION du paquet :

- **INPUT** → paquets À DESTINATION de cette machine (ex : quelqu'un se connecte en SSH sur nous). Chemin : PREROUTING → INPUT
- **OUTPUT** → paquets ÉMIS par cette machine (ex : nous faisons un curl). Chemin : OUTPUT → POSTROUTING
- **FORWARD** → paquets qui TRAVERSENT la machine sans lui être destinés (cas du routeur / du NAT). Chemin : PREROUTING → FORWARD → POSTROUTING
- **PREROUTING** → tout de suite à l'arrivée, avant la décision de routage (sert au DNAT) / **POSTROUTING** → juste avant la sortie (sert au MASQUERADE / SNAT)
- Pour retenir : INPUT = « pour moi », OUTPUT = « de moi », FORWARD = « à travers moi »

Les 2 tables à connaître :

- **filter** → table PAR DÉFAUT quand on n'écrit pas -t : sert à autoriser / bloquer ; chaînes INPUT, FORWARD, OUTPUT
- **nat** → `-t nat` : sert à rediriger / masquer ; chaînes PREROUTING, POSTROUTING, OUTPUT

Actions (-j) :

- **ACCEPT** = laisse passer
- **DROP** = jette en silence (l'émetteur attend puis timeout)
- **REJECT** = refuse avec un message d'erreur
- DNAT, MASQUERADE = voir au-dessus

## iptables : règles

- `sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT` → autorise les connexions entrantes SSH vers cette machine (-A = ajoute à la FIN de la chaîne)
- `sudo iptables -A INPUT -s 10.0.0.0/24 -p tcp --dport 80 -j ACCEPT` → autorise le port 80 seulement depuis ce réseau (-s = source, -d = destination)
- `sudo iptables -A INPUT -p tcp --dport 23 -j DROP` → bloque le port 23 en entrée
- `sudo iptables -A OUTPUT -p tcp --dport 25 -j DROP` → bloque ce que la machine envoie vers un port 25 (sortant)
- `sudo iptables -A FORWARD -s 10.5.5.0/24 -d 192.168.5.2 -p tcp --dport 80 -j ACCEPT` → autorise le trafic routé à travers la machine (utile avec un DNAT si la politique FORWARD est DROP)
- `sudo iptables -I INPUT 1 -s 10.0.0.5 -j DROP` → -I = INSÈRE en position 1 (tout en haut), alors que -A ajoute à la fin
- `sudo iptables -L -n -v --line-numbers` → liste les règles de la table filter avec leurs numéros
- `sudo iptables -D INPUT 3` → supprime la règle n°3 de INPUT
- ⚠️ L'ORDRE COMPTE : le paquet est comparé aux règles de haut en bas, la PREMIÈRE qui correspond s'applique et on s'arrête. Un DROP placé avant un ACCEPT annule l'ACCEPT
- `sudo iptables -P INPUT DROP` → politique par défaut = tout ce qui ne correspond à aucune règle est bloqué. ⚠️ Poser d'abord la règle ACCEPT du port 22, sinon tu te coupes toi-même en SSH
- `-i eth0` = interface d'ENTRÉE (INPUT, FORWARD, PREROUTING) / `-o eth0` = interface de SORTIE (OUTPUT, FORWARD, POSTROUTING)

## iptables : persistance

Valable pour TOUTES les règles ci-dessus (filter et nat) :

- `sudo apt install iptables-persistent` → propose d'enregistrer les règles actuelles (répondre Yes)
- `sudo netfilter-persistent save` → à relancer après toute modification ultérieure (règles stockées dans /etc/iptables/rules.v4)
