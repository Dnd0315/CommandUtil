# LDAP / NSS (comptes centralisés)

[← Index](../README.md)

NSS (Name Service Switch) = décide OÙ le système cherche les utilisateurs, les groupes et les mots de passe : fichiers locaux (/etc/passwd…) et/ou serveur LDAP.

- `/etc/nsswitch.conf` → une ligne par type de donnée : passwd: (utilisateurs) / group: (groupes) / shadow: (mots de passe). Ajouter `ldap` à la fin d'une ligne = aller AUSSI chercher ces données sur le serveur LDAP. Ex : `passwd: files systemd ldap`
- nslcd = « name service local daemon » = le service qui interroge le serveur LDAP → config dans `/etc/nslcd.conf` (⚠️ PAS /etc/ldap/ldap.conf) ; adresse du serveur sur la ligne `uri ldap://IP/`
- `sudo systemctl restart nslcd` → applique les changements de nslcd.conf (« make sure your changes become active »)
- `getent passwd` / `getent group` → liste les utilisateurs / groupes vus par le système (locaux + LDAP) : sert à vérifier
