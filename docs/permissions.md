# Permissions

[← Index](../README.md)

Chaque fichier a un propriétaire (u), un groupe (g) et les autres (o). Droits : r lire, w écrire, x exécuter (pour un dossier : x = pouvoir y entrer).

## Propriétaire et groupe

- `chgrp dev file` → change le groupe propriétaire du fichier en « dev »
- `groups` / `groups john` → affiche les groupes dont fait partie l'utilisateur courant / john
- `chown john file` → change le propriétaire du fichier en john
- `chown john:dev file` → change propriétaire ET groupe en une fois
- `chown -R john:dev dir/` → idem, récursivement sur tout le contenu du dossier

## chmod

- `chmod u+x file` → ajoute le droit d'exécution au propriétaire (+ ajoute, - retire)
- `chmod g-w file` → retire le droit d'écriture au groupe
- `chmod u=rwx,g=rx,o= file` → fixe les droits exacts (o= : aucun droit pour les autres)
- `chmod 640 file` → mode octal : r=4, w=2, x=1, on additionne par catégorie. 640 = rw- r-- ---
- `chmod -R u=rwX,g=rX,o= dir/` → récursif ; le X majuscule met x seulement sur les dossiers (et les fichiers déjà exécutables), pas sur les fichiers ordinaires

## Bits spéciaux

Chiffre placé devant les 3 autres : 4 SUID, 2 SGID, 1 sticky.

- `chmod 4755 file` ou `chmod u+s file` → SUID : le programme s'exécute avec les droits de son PROPRIÉTAIRE (ex : /usr/bin/passwd)
- `chmod 2775 dir` ou `chmod g+s dir` → SGID sur un dossier : les fichiers créés dedans héritent du groupe du dossier (dossiers partagés)
- `chmod 1777 dir` ou `chmod +t dir` → sticky bit : seul le propriétaire d'un fichier peut le supprimer (ex : /tmp)
- `chmod 6755 file` → SUID + SGID (4+2)
- Dans `ls -l` : s/t minuscule = bit spécial + x ; S/T majuscule = bit spécial posé SANS x

## ACL

ACL = droits pour un utilisateur ou groupe précis, en plus de u/g/o.

- `setfacl -m u:john:rw file` → donne lecture + écriture à john uniquement (-m = modify)
- `setfacl -m g:dev:rx dir` → donne lecture + accès au groupe dev
- `setfacl -x u:john file` → retire l'entrée ACL de john
- `setfacl -b file` → supprime toutes les ACL
- `getfacl file` → affiche les ACL (un + à la fin des droits dans `ls -l` signale une ACL)
- `setfacl -R -m g:dev:rwX dir/` → applique récursivement (X = le droit x seulement sur les dossiers et les fichiers déjà exécutables)
- `setfacl -d -m g:dev:rwx dir/` → ACL PAR DÉFAUT (-d) : héritée par tous les fichiers et dossiers créés ensuite dans dir ; `setfacl -k dir/` → les supprime
- Le masque (`mask::` dans getfacl) plafonne les droits effectifs des entrées ACL (ligne `#effective:`) : `setfacl -m m::rwx file` le rouvre

## Liens et infos

- `ln target hardlink` → lien physique : un 2e nom pour le même fichier (même inode). Même système de fichiers, pas de dossiers
- `ln -s target symlink` → lien symbolique : un raccourci vers un chemin. Cassé si la cible est supprimée. Utiliser un chemin absolu pour la cible
- `stat file` → détails : taille, inode, droits en octal, nombre de liens, dates d'accès/modification
- `stat -c '%a %U:%G %n' file` → droits en octal, propriétaire:groupe et nom, sur une seule ligne

## umask (droits par défaut)

- `umask` → affiche le masque actuel (ex : 0022) ; `umask 027` → le change pour la session
- Droits des nouveaux fichiers = 666 moins le masque (0022 → 644) ; des nouveaux dossiers = 777 moins le masque (0022 → 755). Avec 027 : fichiers 640, dossiers 750
- Permanent : dans `~/.bashrc` (un utilisateur), dans `/etc/profile` ou `UMASK` de `/etc/login.defs` (tous)

## Attributs de fichier (chattr)

- `lsattr file` → affiche les attributs étendus (i, a…)
- `sudo chattr +i file` → fichier IMMUABLE : ni modification, ni suppression, ni renommage, même par root ; `sudo chattr -i file` pour lever
- `sudo chattr +a file` → ajout seul (append only) : on peut ajouter à la fin mais rien effacer (pratique pour un log)
