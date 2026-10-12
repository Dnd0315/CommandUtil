# Créer / écrire dans un fichier

[← Index](../README.md)

## Créer et écrire

- `touch file` → crée un fichier vide (ou met à jour sa date s'il existe déjà)
- `mkdir -p /a/b/c` → crée le dossier ET tous les dossiers parents manquants
- `echo "bonjour" > file` → crée le fichier avec ce texte (⚠️ ÉCRASE le contenu s'il existe)
- `echo "ligne 2" >> file` → AJOUTE le texte à la fin, sans rien effacer
- `printf "ligne1\nligne2\n" > file` → plusieurs lignes en une commande (\n = retour à la ligne)
- `cat > file << EOF` → plusieurs lignes : taper les lignes, et terminer par une ligne contenant seulement `EOF`
- `cat file` → vérifier le contenu

## Écrire avec les droits root

- ⚠️ `sudo echo "x" > /etc/file` NE MARCHE PAS : la redirection > est faite par ton shell, qui n'a pas les droits root
- `echo "x" | sudo tee /etc/file` → écrit avec les droits root (écrase) ; `sudo tee -a` = ajoute à la fin (exemple : paramètre noyau persistant → voir [Système](systeme.md))
- `getenforce | sudo tee /opt/selinuxmode.txt` → enregistre le résultat d'une commande (écrit ET affiche)

## Redirections

3 flux : **1 = stdout** (sortie normale) / **2 = stderr** (erreurs) / **0 = stdin**

- `cmd > f` = `cmd 1> f` → stdout dans f (écrase) ; `cmd 2> f` → seulement stderr dans f ; `cmd >> f` → ajoute à la fin
- `cmd > f 2>&1` → stdout ET stderr dans le même fichier (l'ordre compte : > f d'abord, puis 2>&1) ; version courte bash : `cmd &> f` ; `cmd 2>&1 | wc -l` → compte tout (stdout + stderr)
- `cmd > /dev/null` → jette la sortie ; `cmd > /dev/null 2>&1` → jette tout
- `cmd` puis `echo $? > f` → enregistre le code de retour (0 = succès, autre = erreur) de la DERNIÈRE commande. ⚠️ lancer la commande en entier, ne rien exécuter entre les deux, sinon $? change
