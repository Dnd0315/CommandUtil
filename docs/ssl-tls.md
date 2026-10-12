# SSL / TLS

[← Index](../README.md)

Le chiffrement asymétrique utilise une paire de clés : une clé privée et une clé publique. Pour les clés SSH (`ssh-keygen`), voir [SSH](ssh.md).

## Générer une paire de clés

- `openssl genrsa -out myserver.key 2048` → génère la clé privée RSA
- `openssl rsa -in myserver.key -pubout > myserver.pem` → extrait la clé publique correspondante

## Certificat

Certificate Authority (CA) : Symantec, Digicert, Comodo, GlobalSign.

- `openssl req -new -key myserver.key -out myserver.csr -subj "/C=UAE/ST=DU/O=TLT, Inc./CN=my-server.com"` → génère un Certificate Signing Request (CSR) à envoyer à la CA
- `openssl req -newkey rsa:2048 -nodes -keyout k.key -x509 -days 365 -out c.crt` → crée une clé privée (k.key, non chiffrée grâce à -nodes) et un certificat auto-signé valable 1 an (c.crt)
- `openssl x509 -in c.crt -text -noout` → affiche le contenu d'un certificat
