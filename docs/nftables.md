# nftables

[← Index](../README.md)

nftables = le successeur d'iptables (même rôle : filtrer, rediriger, masquer). Concepts d'iptables (voir [Réseau](reseau.md)) : tables, chaînes, règles, politique par défaut ; ici, tables et chaînes sont créées par vous.

- Famille de table : `ip` (IPv4), `ip6` (IPv6), `inet` (IPv4 + IPv6 : le plus pratique), `bridge`, `arp`
- Une chaîne « de base » s'accroche à un point du parcours des paquets : `hook input` / `forward` / `output` / `prerouting` / `postrouting`

## Voir

- `sudo apt install nftables` → installation
- `sudo nft list ruleset` → toutes les tables, chaînes et règles
- `sudo nft list tables` → liste des tables ; `sudo nft list chain inet filter input` → une chaîne
- `sudo nft -a list ruleset` → affiche aussi le « handle » (numéro) de chaque règle, utile pour la supprimer

## Créer table, chaîne, règles

- `sudo nft add table inet filter` → crée la table filter (famille inet)
- `sudo nft add chain inet filter input '{ type filter hook input priority 0 ; policy accept ; }'` → crée la chaîne input accrochée à l'entrée (les quotes protègent { } et ; du shell). `policy drop` = tout bloquer par défaut (⚠️ autoriser SSH d'abord)
- `sudo nft add rule inet filter input tcp dport 22 accept` → autorise SSH en entrée
- `sudo nft add rule inet filter input ip saddr 10.0.0.0/24 tcp dport 80 accept` → port 80 seulement depuis ce réseau
- `sudo nft add rule inet filter input tcp dport 23 drop` → bloque le port 23
- `sudo nft add rule inet filter input ct state established,related accept` → laisse passer les réponses aux connexions déjà établies
- `sudo nft add rule inet filter input iif lo accept` → autorise l'interface locale (lo)
- `sudo nft insert rule inet filter input tcp dport 8080 accept` → insère en TÊTE de chaîne (add ajoute à la fin ; l'ordre compte, comme avec iptables)
- `sudo nft delete rule inet filter input handle 5` → supprime la règle de handle 5
- `sudo nft flush ruleset` → efface TOUT (⚠️ en SSH avec une politique drop, on se coupe)

## NAT (redirection de port, masquerade)

```bash
sudo nft add table ip nat
sudo nft add chain ip nat prerouting '{ type nat hook prerouting priority -100 ; }'
sudo nft add chain ip nat postrouting '{ type nat hook postrouting priority 100 ; }'
sudo nft add rule ip nat prerouting tcp dport 81 dnat to 192.168.5.2:80
sudo nft add rule ip nat prerouting tcp dport 6000 redirect to :6001
sudo nft add rule ip nat postrouting ip saddr 10.5.5.0/24 masquerade
```

- `dnat to IP:port` → vers une autre machine ; `redirect to :port` → vers un port local ; `masquerade` → remplace la source par l'adresse de sortie
- Pour que la machine route vraiment : `net.ipv4.ip_forward=1` (voir [Système](systeme.md))

## Persistance

Le service nftables charge `/etc/nftables.conf` au démarrage. Fichier complet (la ligne `flush ruleset` évite d'empiler les règles à chaque rechargement) :

```
#!/usr/sbin/nft -f
flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        iif lo accept
        ct state established,related accept
        tcp dport 22 accept
        ip saddr 10.0.0.0/24 tcp dport 80 accept
    }
    chain forward {
        type filter hook forward priority 0; policy drop;
    }
    chain output {
        type filter hook output priority 0; policy accept;
    }
}
```

- `sudo nft list ruleset | sudo tee /etc/nftables.conf` → exporte les règles actuelles dans le fichier (ajouter ensuite `flush ruleset` en 1re ligne)
- `sudo nft -c -f /etc/nftables.conf` → vérifie la syntaxe SANS appliquer
- `sudo nft -f /etc/nftables.conf` → charge le fichier
- Activer au démarrage : `enable --now nftables` (voir [systemd](systemd.md))

## Passer d'iptables à nftables

- `iptables-translate -A INPUT -p tcp --dport 22 -j ACCEPT` → affiche la commande nft équivalente à une règle iptables (sans rien appliquer)
