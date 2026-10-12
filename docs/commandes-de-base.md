# Commandes de base

[← Index](../README.md)

Créer des fichiers et dossiers (`touch`, `mkdir -p`) : voir [Créer / écrire dans un fichier](fichiers-et-redirections.md). Liens et droits : voir [Permissions](permissions.md).

## Naviguer et lister

- `pwd` → dossier courant
- `cd /etc` → va dans /etc ; `cd` ou `cd ~` → home ; `cd -` → retourne au dossier précédent ; `cd ..` → dossier parent
- `ls -lah` → liste détaillée (-l), avec fichiers cachés (-a), tailles lisibles (-h)
- `ls -ld /etc` → propriétés du DOSSIER lui-même (sans lister son contenu)
- `ls -lt` → triés par date de modification (récents en premier) ; `ls -ltr` → inverse (récents en dernier)
- `ls -R` → récursif ; `ls -li` → avec les numéros d'inode (repérer les liens physiques)
- `tree -L 2 /etc` → arborescence sur 2 niveaux (`sudo apt install tree`)

## Copier, déplacer, supprimer

- `cp file dest` → copie ; `cp -r dir dest` → copie un dossier (récursif) ; `cp -i` demande avant d'écraser
- `cp -a dir dest` → copie « archive » : conserve droits, propriétaires, dates et liens (pour sauvegarder ou cloner un dossier)
- `mv src dest` → déplace OU renomme (`mv ancien nouveau`) ; `mv -i` demande avant d'écraser
- `rm file` → supprime un fichier ; `rm -r dir` → supprime un dossier et son contenu ; `rm -i` demande confirmation
- ⚠️ `rm -rf dir` → supprime sans rien demander, définitif : relire la commande avant Entrée
- `rmdir dir` → supprime un dossier VIDE seulement (sécurité)
- `readlink -f lien` → chemin absolu réel d'un lien symbolique
- `basename /etc/nginx/nginx.conf` → nginx.conf / `dirname /etc/nginx/nginx.conf` → /etc/nginx

## Lire un fichier

- `less file` → lit page par page : Espace page suivante, `b` précédente, `g` début, `G` fin, `/motif` cherche (puis `n` suivant), `F` suit comme tail -f, `q` quitte. `more` = version plus limitée
- `file nom` → type réel du contenu (texte, exécutable, archive…), sans se fier à l'extension

## Retrouver une commande ou un fichier

- `type ls` → dit si c'est un alias, une fonction, une commande interne du shell ou un programme (et où)
- `whereis nginx` → binaire, sources et pages de manuel (`which` : voir [Logs](logs.md))
- `locate nginx.conf` → recherche instantanée dans une base de données (`sudo apt install plocate`) ; `locate -i` ignore la casse
- `sudo updatedb` → met à jour cette base (sinon les fichiers récents sont introuvables avec locate)
- Recherche en direct sur le disque (plus lente, plus précise) : voir [find](find.md)

## Documentation

- `man ls` → manuel d'une commande (`q` quitte, `/motif` cherche, `n` résultat suivant)
- `man 5 passwd` → section précise du manuel : 1 commandes, 5 formats de fichiers de configuration, 8 administration système. `man passwd` ouvre la section 1 ; `man 5 passwd` explique le FICHIER /etc/passwd
- `man -k mot` = `apropos mot` → cherche une commande ou un fichier de config par mot-clé (ex : `man -k nslcd`)
- `whatis ls` → description en une ligne
- `info coreutils` → documentation GNU détaillée (navigation : `n`, `p`, `u`, `q`)
- `commande --help` → aide rapide des options
- `/usr/share/doc/PAQUET/` → documentation et exemples de configuration installés avec un paquet

## Historique et raccourcis du shell

- `history` → commandes tapées ; `history | grep ssh` → retrouver une commande
- `!!` → relance la dernière commande ; `sudo !!` → la relance avec sudo ; `!123` → relance la commande n°123 de l'historique
- `Ctrl+R` → recherche incrémentale dans l'historique ; `Ctrl+C` interrompt ; `Ctrl+D` fin d'entrée / déconnexion ; `Ctrl+L` efface l'écran ; `Ctrl+A` / `Ctrl+E` début / fin de ligne ; `Ctrl+U` efface jusqu'au début de ligne
- `Tab` complète un nom ; deux fois `Tab` liste les choix

## Divers utiles

- `date +%F` → date au format 2026-10-12 ; `date +%F_%H%M%S` → date et heure pour nommer un fichier de sauvegarde
- `seq 1 5` → affiche 1 à 5 (une par ligne) ; `sleep 5` → attend 5 secondes
- `time commande` → mesure la durée d'exécution
- `watch -n 2 'df -h'` → relance la commande toutes les 2 secondes et rafraîchit l'écran
