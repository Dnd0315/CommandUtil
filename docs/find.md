# find

[← Index](../README.md)

Syntaxe : `find [où chercher] [critères] [action]`

## Critères

- `find /usr/share/ -name '*.jpg'` → fichiers dont le nom finit par .jpg (-iname = sans tenir compte de la casse)
- `find /var -type f` → uniquement les fichiers (d = dossiers, l = liens)
- `find /var -size +10M` → plus de 10 Mo ; -10M = moins de 10 Mo ; unités k, M, G
- `find / -perm -4000 2>/dev/null` → fichiers avec SUID ; 2>/dev/null cache les erreurs « Permission denied »
- `find / -perm 644` → droits EXACTEMENT 644
- `find / -perm -644` → AU MOINS tous ces droits
- `find / -perm /222` → AU MOINS UN de ces droits (ici : écriture pour quelqu'un)
- `find /home -user john -group dev` → appartenant à john et au groupe dev
- `find . -mmin -60` → modifiés il y a moins de 60 minutes
- `find . -mtime +7` → modifiés il y a plus de 7 jours
- `find . -type f ! -newermt "2020-01-01"` → fichiers modifiés AVANT le 1er janvier 2020 (-newermt = plus récent que cette date ; ! l'inverse). Toujours tester sans -exec (puis `| wc -l`), puis ajouter `-exec rm {} \;`
- `find . -maxdepth 1 -type f -size -3k` → seulement le dossier courant, sans descendre dans les sous-dossiers (-maxdepth 1) ; -size -3k = moins de 3 Ko ; -size +10k = plus de 10 Ko

## Actions

- `find . -name '*.log' -exec rm {} \;` → exécute rm sur chaque résultat ({} = le fichier trouvé, \; = fin de commande)
- `find . -name '*.txt' -exec cp {} /backup/ \;` → copie chaque résultat dans /backup
- `find . -maxdepth 1 -type f -perm 777 -exec mv {} ./compromised \;` → déplace chaque fichier trouvé dans le dossier compromised (créer small/, large/, compromised/ avant ; -maxdepth 1 évite de reparcourir les sous-dossiers)
- `find . -name '*.log' -exec echo {} +` → avec + : UNE seule commande avec tous les fichiers (echo f1 f2 f3) ; avec \; : une commande par fichier
- `find /tmp -empty -delete` → trouve et supprime les fichiers/dossiers vides
- Autre façon de traiter les résultats : les passer à `xargs` (voir [Texte](texte.md))

## Combiner des critères

- `find /home -type f -user john -size +10M` → critères à la suite = ET : fichiers de john ET de plus de 10 Mo (TOUS les critères doivent être vrais)
- `find /var/log -name '*.log' -o -name '*.txt'` → -o = OU : .log OU .txt (au moins UN critère vrai)
- `find /srv -not -user root` = `find /srv ! -user root` → NON : inverse le critère qui suit (fichiers qui n'appartiennent PAS à root)
- `find /etc -type f -not -name '*.conf'` → fichiers ET dont le nom ne finit PAS par .conf
- `find /var -type f \( -name '*.log' -o -name '*.txt' \)` → ET + OU mélangés : parenthèses OBLIGATOIRES, échappées avec \ et entourées d'espaces. Sans elles, find comprend « (fichiers ET .log) OU .txt » et récupère aussi des dossiers nommés .txt
