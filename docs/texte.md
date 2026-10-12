# Texte

[← Index](../README.md)

## sed

- `sed 's/canda/canada/g' file.txt` → remplace canda par canada et AFFICHE le résultat (fichier inchangé)
- `sed -i 's/canda/canada/g' file.txt` → idem mais modifie le fichier (-i = in-place ; s = substitute ; g = toutes les occurrences de la ligne, sinon seulement la 1re)
- `sed -n '5,10p' file` → affiche seulement les lignes 5 à 10. p = print (afficher) ; -n = désactive l'affichage automatique de toutes les lignes. Sans -n, les lignes 5 à 10 apparaissent EN DOUBLE au milieu du fichier complet → toujours -n avec p
- `sed -n '3p' file` → seulement la ligne 3 / `sed -n '$p' file` → seulement la dernière ligne
- `sed -n '/error/p' file` → seulement les lignes contenant « error » (comme grep)
- `sed '5,10d' file` → d = delete : affiche tout SAUF les lignes 5 à 10 (pas de -n ici : on garde l'affichage automatique)
- `sed '/^#/d' file` → supprime les lignes qui commencent par # (commentaires)
- `sed -i.bak 's/old/new/g' file` → modifie le fichier ET garde une copie de sauvegarde file.bak
- `sed -i '3i ligne insérée' file` → insère une ligne AVANT la ligne 3 (`3a` : après)
- `sed -E 's/([0-9]+)-([0-9]+)/\2-\1/' file` → regex étendues (-E) et groupes : \1, \2 réutilisent ce qui a été capturé entre parenthèses
- `sed -e 's/a/b/' -e 's/c/d/' file` → plusieurs commandes à la suite

## cut

- `cut -d ',' -f 3 file.txt` → extrait la 3e colonne d'un fichier séparé par des virgules (-d = délimiteur, UN seul caractère ; -f = numéro de champ ; -f 1,3 = plusieurs)
- `cut -d ',' -f 3 file.txt > countries.txt` → > écrit le résultat dans un fichier (écrase) ; >> ajoute à la fin

## awk

awk = lit un fichier ligne par ligne et découpe chaque ligne en colonnes ($1, $2…). Plus puissant que cut : le séparateur par défaut est « un ou plusieurs espaces ».

- `awk '{print $1}' file` → affiche la 1re colonne de chaque ligne ($0 = la ligne entière, $NF = la dernière colonne)
- `awk -F ':' '{print $1}' /etc/passwd` → -F = séparateur (ici :) ; affiche les noms d'utilisateurs
- `awk -F ':' '{print $1, $7}' /etc/passwd` → plusieurs colonnes (la virgule met un espace entre les deux)
- `awk -F ':' '{print $1 " → " $7}' /etc/passwd` → texte ajouté entre guillemets doubles
- `awk '/error/ {print $0}' file` → seulement les lignes qui contiennent « error » (comme grep)
- `awk '$3 > 100' file` → seulement les lignes dont la 3e colonne est supérieure à 100
- `awk -F ':' '$3 >= 1000 {print $1}' /etc/passwd` → utilisateurs dont l'UID est ≥ 1000
- `awk 'NR==5' file` → seulement la ligne 5 (NR = numéro de la ligne) ; `awk 'NR>=5 && NR<=10' file` → lignes 5 à 10
- `awk '{sum += $2} END {print sum}' file` → additionne la 2e colonne ; END = exécuté une fois à la fin
- `awk 'END {print NR}' file` → compte les lignes (comme wc -l)
- `awk '{print $1}' file | sort | uniq -c` → compte les occurrences de chaque valeur de la colonne 1
- `awk -v seuil=100 '$3 > seuil' file` → passe une valeur à awk avec -v (variable utilisable dans le programme)
- `awk '{s[$1] += $2} END {for (k in s) print k, s[k]}' file` → somme de la colonne 2 pour chaque valeur de la colonne 1 (tableau associatif ; ordre d'affichage non garanti)
- `awk 'length > 80' file` → lignes de plus de 80 caractères
- `awk 'BEGIN {FS=":"} {print $1}' /etc/passwd` → BEGIN = exécuté avant de lire le fichier ; FS = séparateur (équivalent de -F)

## sort, uniq, wc, head, tail

- `sort file` → trie les lignes par ordre alphabétique
- `sort -n` → tri numérique / `sort -r` → ordre inverse / `sort -k2` → trie selon la 2e colonne
- `uniq file` → supprime les doublons, mais seulement s'ils sont COLLÉS l'un à l'autre
- `sort file | uniq` = `sort -u file` → trie puis supprime tous les doublons
- `sort file | uniq -c` → compte les occurrences de chaque ligne
- `wc -l file` → compte les lignes (-w mots, -c octets)
- `head -n 5 file` / `tail -n 5 file` → 5 premières / 5 dernières lignes
- `tail -f file` → suit le fichier en direct (logs) ; Ctrl+C pour sortir
- `tail -n +2 file` → tout SAUF la 1re ligne (à partir de la ligne 2) : saute un en-tête
- `sort -t ':' -k3 -n /etc/passwd` → trie /etc/passwd par UID (-t séparateur, -k3 3e champ, -n numérique)
- `uniq -d file` → affiche seulement les lignes en double / `uniq -u file` → seulement les lignes uniques (fichier trié)

## diff

- `diff file1 file2` → montre les différences entre deux fichiers
- `diff -c file1 file2` → format « contexte » : affiche les lignes autour des changements
- `diff -y file1 file2` = `sdiff file1 file2` → affichage côte à côte
- `diff -u old new` → format « unifié » (celui des patchs, lignes -/+) / `diff -r dir1 dir2` → compare deux dossiers récursivement
- `cmp file1 file2` → compare octet par octet (fichiers binaires) : indique la première différence
- `sha256sum file` → empreinte du fichier (`md5sum` : plus ancien) ; `sha256sum -c SHA256SUMS` → vérifie des fichiers à partir d'une liste d'empreintes (« OK » / « FAILED »)

## grep

- `grep 'word' file` → affiche les lignes contenant « word »
- `grep -i` → ignore la casse (Word = word)
- `grep -r 'word' folder/` → cherche dans tous les fichiers du dossier, récursivement
- `grep -v` → affiche les lignes qui NE contiennent PAS le motif
- `grep -w` → mot entier seulement (« cat » ne trouve pas « category »)
- `grep -o` → affiche seulement la partie qui correspond, pas toute la ligne
- `grep -c` → compte les lignes trouvées / `grep -n` → affiche les numéros de ligne
- `grep -E` = `egrep` → active les regex étendues (voir [Regex](regex.md))
- `grep -l 'motif' *` → affiche seulement les NOMS des fichiers qui contiennent le motif (`-L` : ceux qui ne le contiennent pas)
- `grep -q 'motif' file` → silencieux : n'affiche rien, seul le code de retour compte (0 = trouvé) : pour les scripts
- `grep -A 2 'motif' file` → + 2 lignes APRÈS chaque résultat / `-B 2` → avant / `-C 2` → autour
- `grep -e 'a' -e 'b' file` → l'un OU l'autre motif ; `grep -f motifs.txt file` → motifs lus dans un fichier
- `grep -r --include='*.conf' 'motif' /etc` → recherche récursive limitée à certains fichiers

## xargs

- `find . -name '*.tmp' | xargs rm` → transforme les lignes reçues en ARGUMENTS d'une commande (alternative à `-exec`)
- `find . -name '*.log' -print0 | xargs -0 rm` → -print0 / -0 : séparateur nul, indispensable si des noms contiennent des espaces
- `xargs -n 1 echo` → une exécution par argument (-n 1) ; `xargs -I {} cp {} /backup/` → place chaque argument à l'endroit de {}

## Transformer et assembler

- `echo salut | tr 'a-z' 'A-Z'` → convertit des caractères (tr lit l'entrée standard, pas un nom de fichier) ; `tr -d '\r'` supprime les retours chariot Windows ; `tr -s ' '` compresse les espaces répétés
- `paste f1 f2` → colle les lignes côte à côte (séparées par une tabulation, `-d ','` change le séparateur) ; `paste -sd, file` → toutes les lignes sur une seule, séparées par des virgules
- `join -t ',' f1 f2` → jointure sur le 1er champ de chaque fichier (fichiers triés)
- `comm -12 f1 f2` → lignes communes à deux fichiers triés ; `comm -23 f1 f2` → lignes seulement dans f1
- `nl file` (ou `cat -n file`) → numérote les lignes ; `tac file` → inverse l'ordre des lignes ; `rev` → inverse chaque ligne
- `column -t -s ':' /etc/passwd` → aligne en colonnes (séparateur : )
- `split -l 1000 file part_` → découpe en morceaux de 1000 lignes (part_aa, part_ab…) ; `-b 10M` découpe par taille
