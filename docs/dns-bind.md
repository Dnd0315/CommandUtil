# Serveur DNS (BIND9)

[← Index](../README.md)

Deux rôles possibles pour un serveur DNS :

- Serveur CACHE / résolveur (« caching DNS server ») : répond aux clients du réseau en interrogeant d'autres serveurs (ou en descendant la hiérarchie) et mémorise les réponses
- Serveur FAISANT AUTORITÉ sur une ou plusieurs zones (« maintain a DNS zone ») : détient les enregistrements d'un domaine (www.example.lan → 10.0.0.10). Un serveur peut jouer les deux rôles

Côté client (résolution, `dig`, `/etc/resolv.conf`) : voir [Réseau](reseau.md) et [Dépannage réseau](depannage-reseau.md).

## Installation et fichiers

- `sudo apt install bind9 bind9-utils bind9-dnsutils` → serveur, outils de contrôle (named-checkconf, rndc…) et `dig` (RHEL : paquets `bind` et `bind-utils`). Service : `bind9` (Debian / Ubuntu) ou `named` (RHEL) : voir [systemd](systemd.md)
- Debian / Ubuntu : `/etc/bind/named.conf` (inclut les autres), `/etc/bind/named.conf.options` (options générales), `/etc/bind/named.conf.local` (VOS zones), fichiers de zone dans `/etc/bind/` (modèles `db.*`) ; les zones esclaves s'écrivent dans `/var/cache/bind/`
- RHEL : tout dans `/etc/named.conf`, fichiers de zone dans `/var/named/`
- Commentaires dans named.conf : `//`, `#` ou `/* … */` ; dans un fichier de zone : `;`. Chaque instruction de named.conf finit par `;`

## Serveur cache avec redirecteurs

Dans `/etc/bind/named.conf.options` :

```
acl "lan" { 10.0.0.0/24; };

options {
    directory "/var/cache/bind";
    listen-on { 127.0.0.1; 10.0.0.5; };
    allow-query { localhost; lan; };
    recursion yes;
    allow-recursion { localhost; lan; };
    forwarders { 8.8.8.8; 1.1.1.1; };
    dnssec-validation auto;
};
```

- `listen-on` → adresses d'écoute (IPv6 : `listen-on-v6`) / `allow-query` → qui peut poser des questions / `recursion yes` + `allow-recursion` → qui a droit à la résolution récursive (ne JAMAIS l'ouvrir à Internet entier)
- `forwarders { … }` → transmet les requêtes à ces serveurs au lieu de résoudre soi-même (`forward only;` interdit de résoudre soi-même si les redirecteurs ne répondent pas ; par défaut `forward first`)
- `acl "lan" { … };` → donne un nom à un groupe d'adresses, réutilisable dans les autres directives
- Tester depuis un client avec `dig @10.0.0.5 example.com` (voir [Dépannage réseau](depannage-reseau.md)) : le 2e appel est servi par le cache (« Query time: 0 msec »)

## Vérifier et recharger

- `sudo named-checkconf` → vérifie la syntaxe de named.conf et des fichiers inclus (silence = OK) ; `sudo named-checkconf -z` → charge aussi les zones pour les contrôler
- `sudo named-checkzone example.lan /etc/bind/db.example.lan` → vérifie un fichier de zone : affiche le numéro de série et « OK »
- `sudo rndc reload` → recharge la config et les zones sans redémarrer ; `sudo rndc reload example.lan` → une seule zone
- `sudo rndc flush` → vide le cache / `sudo rndc status` → état du serveur / `sudo rndc querylog on` → journalise chaque requête reçue (`off` pour arrêter)
- `sudo systemctl restart bind9` → redémarrage complet (RHEL : `named`)
- Logs : `journalctl -u bind9` (voir [Logs](logs.md)) ; une erreur de zone y apparaît au chargement
- Pare-feu : le DNS écoute sur le port 53 en UDP ET en TCP (transferts de zone, réponses longues) : ouvrir les deux (voir [Réseau](reseau.md) ; RHEL : service `dns` de firewalld, voir [Équivalents RHEL](rhel-fedora.md))

## Zone directe (maître)

Déclarer la zone dans `/etc/bind/named.conf.local` :

```
zone "example.lan" {
    type master;
    file "/etc/bind/db.example.lan";
    allow-transfer { 10.0.0.6; };
    notify yes;
};
```

(`type primary;` dans les BIND récents : `master` fonctionne partout.) Fichier de zone `/etc/bind/db.example.lan` :

```
$TTL 86400
@       IN  SOA  ns1.example.lan. admin.example.lan. (
                 2026101201  ; serial
                 3600        ; refresh
                 900         ; retry
                 604800      ; expire
                 86400 )     ; durée de cache des réponses négatives
@       IN  NS     ns1.example.lan.
@       IN  MX 10  mail.example.lan.
@       IN  TXT    "v=spf1 mx -all"
ns1     IN  A      10.0.0.5
mail    IN  A      10.0.0.7
www     IN  A      10.0.0.10
app     IN  CNAME  www
v6host  IN  AAAA   2001:db8::10
```

- Un nom SANS point final est complété avec le nom de la zone (`www` = www.example.lan.) ; un nom COMPLET se termine par un point (`ns1.example.lan.`). Oublier le point final est l'erreur classique
- `@` = le nom de la zone ; `$TTL` = durée de vie par défaut des enregistrements (secondes)
- SOA : serveur principal, adresse du responsable (`admin.example.lan.` = admin@example.lan), puis serial / refresh / retry / expire / TTL négatif
- ⚠️ Le serial doit AUGMENTER à chaque modification (convention AAAAMMJJnn), sinon les serveurs secondaires ne se mettent pas à jour
- Types : `A` IPv4 / `AAAA` IPv6 / `CNAME` alias vers un autre nom (jamais à la racine de la zone, ni avec d'autres enregistrements) / `MX` serveur de messagerie (plus petite priorité = préféré) / `NS` serveur de noms de la zone (obligatoire) / `TXT` texte libre (SPF…) / `PTR` résolution inverse
- Après modification : augmenter le serial, `named-checkzone`, puis `rndc reload example.lan`, puis tester : `dig @localhost www.example.lan +short`

## Zone inverse (IP → nom)

Pour 10.0.0.x, la zone s'appelle `0.0.10.in-addr.arpa` (les 3 premiers octets à l'envers). Dans named.conf.local :

```
zone "0.0.10.in-addr.arpa" {
    type master;
    file "/etc/bind/db.10.0.0";
};
```

Fichier `/etc/bind/db.10.0.0` :

```
$TTL 86400
@    IN  SOA  ns1.example.lan. admin.example.lan. ( 2026101201 3600 900 604800 86400 )
@    IN  NS   ns1.example.lan.
5    IN  PTR  ns1.example.lan.
10   IN  PTR  www.example.lan.
```

Test : `dig -x 10.0.0.10 @localhost +short` → www.example.lan.

## Serveur secondaire (esclave)

Sur la 2e machine, dans named.conf.local :

```
zone "example.lan" {
    type slave;
    masters { 10.0.0.5; };
    file "/var/cache/bind/db.example.lan";
};
```

(`type secondary;` et `primaries { … };` dans les BIND récents.) Le maître doit autoriser le transfert (`allow-transfer { IP du secondaire; };`).

- Vérifier la synchronisation : comparer les numéros de série avec `dig @10.0.0.5 example.lan SOA +short` et `dig @10.0.0.6 example.lan SOA +short`
- `dig @10.0.0.5 example.lan AXFR` → demande un transfert complet de zone (refusé si allow-transfer ne l'autorise pas : c'est voulu, ne jamais l'ouvrir à tous)

## Problèmes fréquents

- Zone non chargée : lire `journalctl -u bind9` ; erreurs courantes : point final oublié, serial non incrémenté, `;` manquant dans named.conf, enregistrement NS manquant
- Debian / Ubuntu : named tourne sous l'utilisateur `bind` et est confiné par AppArmor : un fichier de zone placé hors de /etc/bind ou /var/cache/bind est refusé (autoriser le chemin dans `/etc/apparmor.d/local/usr.sbin.named`, voir [SELinux / AppArmor](selinux-apparmor.md))
- RHEL : mettre les zones dans /var/named, et vérifier les badges SELinux avec `restorecon` (voir [SELinux / AppArmor](selinux-apparmor.md))
- Un client ne résout pas : `dig @IP_du_serveur` direct (voir [Dépannage réseau](depannage-reseau.md)) pour séparer « serveur en panne » de « client mal configuré » ; vérifier `allow-query` et le pare-feu

## Alternative légère : dnsmasq

Cache DNS + DHCP simples pour un petit réseau ; il lit aussi le `/etc/hosts` du serveur comme enregistrements locaux.

- `sudo apt install dnsmasq` → installation ; config dans `/etc/dnsmasq.conf` ou `/etc/dnsmasq.d/*.conf`
- `server=8.8.8.8` → redirecteur (`no-resolv` ignore /etc/resolv.conf) / `cache-size=1000` → taille du cache
- `address=/www.example.lan/10.0.0.10` → enregistrement local / `local=/example.lan/` → ce domaine n'est résolu QUE localement / `domain=example.lan` + `expand-hosts` → complète les noms de /etc/hosts avec le domaine
- `listen-address=10.0.0.5` avec `bind-interfaces` → écoute seulement sur cette adresse
- `dnsmasq --test` → vérifie la syntaxe de la configuration
- ⚠️ Sur Ubuntu, systemd-resolved occupe déjà le port 53 sur 127.0.0.53 : voir qui écoute avec `ss -ltunp` (voir [Réseau](reseau.md)) ; `bind-interfaces` évite le conflit, ou mettre `DNSStubListener=no` dans /etc/systemd/resolved.conf
