# PAM (Pluggable Authentication Modules)

[← Index](../README.md)

PAM = couche commune d'authentification : les services (login, sshd, sudo, su, passwd) appellent des modules PAM pour authentifier, autoriser, ouvrir une session ou changer un mot de passe. Un fichier par service dans `/etc/pam.d/`.

⚠️ Une erreur dans PAM peut vous enfermer dehors : garder une session root ouverte pendant les tests, et ne la fermer qu'après avoir validé la connexion dans un 2e terminal.

## Lire un fichier /etc/pam.d/

Format d'une ligne : `type  contrôle  module  [arguments]`

- Types : `auth` (qui êtes-vous ?) / `account` (compte autorisé : expiré, verrouillé, horaires ?) / `password` (changement de mot de passe) / `session` (actions à l'ouverture / fermeture de session)
- Contrôles : `required` (doit réussir, mais continue la liste) / `requisite` (doit réussir, sinon arrêt immédiat) / `sufficient` (si réussi, accepte sans lire la suite) / `optional` (sans effet sur le résultat)
- `@include common-auth` → inclut un fichier commun (Debian / Ubuntu : common-auth, common-account, common-password, common-session ; RHEL : system-auth, password-auth)

## Modules courants

- `session required pam_limits.so` → applique /etc/security/limits.conf à la session (voir [Utilisateurs](utilisateurs.md)) : sans cette ligne, limits.conf est ignoré
- `password requisite pam_pwquality.so retry=3 minlen=12` → impose la complexité des mots de passe (paquet libpam-pwquality). Réglages dans `/etc/security/pwquality.conf` : `minlen = 12` (longueur), `minclass = 3` (types de caractères), `dcredit = -1` (au moins 1 chiffre), `ucredit = -1` (majuscule), `lcredit = -1` (minuscule), `ocredit = -1` (caractère spécial), `retry = 3`
- `auth required pam_wheel.so group=sudo use_uid` → dans /etc/pam.d/su : seuls les membres du groupe indiqué peuvent faire `su`
- `account required pam_access.so` → applique /etc/security/access.conf. Format `permission : utilisateurs : origines`, ex : `- : john : ALL` (refuse john) ou `+ : root : LOCAL`
- `account required pam_nologin.so` → si le fichier `/etc/nologin` existe, seul root peut se connecter (le contenu du fichier est affiché aux autres)
- `session required pam_mkhomedir.so skel=/etc/skel umask=0077` → crée le home à la 1re connexion (précieux avec des comptes LDAP, voir [LDAP / NSS](ldap-nss.md))
- `sudo pam-auth-update` → menu Debian / Ubuntu pour activer / désactiver des profils PAM ; `sudo pam-auth-update --enable mkhomedir` → active la création du home
- RHEL / Fedora : `authselect` remplace pam-auth-update (ex : `sudo authselect enable-feature with-mkhomedir`)

## Verrouillage après échecs de connexion (pam_faillock)

Exemple de pile `auth` dans /etc/pam.d/common-auth (Debian / Ubuntu) ou system-auth / password-auth (RHEL) :

```
auth     required       pam_faillock.so preauth silent deny=3 unlock_time=600
auth     sufficient     pam_unix.so nullok try_first_pass
auth     [default=die]  pam_faillock.so authfail deny=3 unlock_time=600
account  required       pam_faillock.so
```

- deny=3 → verrouille après 3 échecs ; unlock_time=600 → déverrouillage automatique après 600 s. Ces valeurs peuvent aussi se mettre dans `/etc/security/faillock.conf`
- `faillock --user john` → affiche les échecs enregistrés pour john
- `sudo faillock --user john --reset` → déverrouille john tout de suite

## Diagnostiquer

- `grep -i pam /var/log/auth.log` → refus et erreurs PAM (RHEL : `/var/log/secure`)
