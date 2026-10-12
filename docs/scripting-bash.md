# Scripts bash

[← Index](../README.md)

## Créer et lancer un script

- Première ligne (shebang) : `#!/bin/bash` → indique l'interpréteur
- Rendre exécutable : `chmod u+x script.sh` (voir [Permissions](permissions.md)) puis `./script.sh`
- `bash script.sh` → lance sans avoir besoin du droit d'exécution
- `bash -x script.sh` → mode trace : affiche chaque commande avant de l'exécuter (déboguer)
- `bash -n script.sh` → vérifie seulement la syntaxe, sans exécuter
- `# texte` → commentaire

## Variables

- `nom="Jean"` → définit une variable (⚠️ AUCUN espace autour du =) ; `echo "$nom"` ou `echo "${nom}_fichier"` → l'utilise
- `date_du_jour=$(date +%F)` → stocke la SORTIE d'une commande (substitution de commande)
- `total=$((2 + 3))` → calcul entier
- `read -p "Votre nom : " nom` → demande une saisie au clavier ; `read -s mdp` = saisie masquée
- Pour que les programmes lancés par le script voient une variable, il faut l'exporter (voir [Variables d'environnement](variables-environnement.md))
- Toujours mettre les variables entre guillemets : `"$nom"` (sinon les espaces cassent la commande)

## Arguments et code de retour

- `$0` nom du script / `$1`, `$2`… arguments / `$#` nombre d'arguments / `$@` tous les arguments
- `$?` → code de retour de la dernière commande (voir [Créer / écrire dans un fichier](fichiers-et-redirections.md))
- `exit 0` → termine le script en succès ; `exit 1` → en échec
- `[ $# -lt 1 ] && { echo "usage : $0 fichier" >&2; exit 1; }` → quitte avec un message d'erreur (sur stderr) si aucun argument n'est donné
- `[ "$(id -u)" -ne 0 ] && { echo "lancer en root" >&2; exit 1; }` → refuse de tourner sans les droits root
- `cmd1 && cmd2` → cmd2 SEULEMENT si cmd1 réussit / `cmd1 || cmd2` → cmd2 SEULEMENT si cmd1 échoue / `cmd1 ; cmd2` → dans tous les cas

## Tests (dans [ ... ] avec des espaces autour)

- Fichiers : `-e` existe / `-f` fichier ordinaire / `-d` dossier / `-s` non vide / `-r` `-w` `-x` lisible, modifiable, exécutable / `-L` lien symbolique
- Chaînes : `"$a" = "$b"` égales / `!=` différentes / `-z "$a"` vide / `-n "$a"` non vide
- Nombres : `-eq` égal / `-ne` différent / `-lt` inférieur / `-le` inférieur ou égal / `-gt` supérieur / `-ge` supérieur ou égal
- `!` inverse un test, `-a` ET, `-o` OU ; `[[ ... ]]` (bash) accepte aussi `&&`, `||` et les motifs (`[[ $f == *.log ]]`)

## Conditions

```bash
if [ -f /etc/passwd ]; then
    echo "le fichier existe"
elif [ -d /etc/passwd ]; then
    echo "c'est un dossier"
else
    echo "absent"
fi
```

```bash
case "$1" in
    start) echo "démarrage" ;;
    stop)  echo "arrêt" ;;
    *)     echo "usage : $0 start|stop" ;;
esac
```

## Boucles

```bash
for i in 1 2 3; do
    echo "valeur $i"
done

for f in /var/log/*.log; do
    echo "$f : $(wc -l < "$f") lignes"
done

for i in $(seq 1 5); do echo "$i"; done
```

```bash
n=1
while [ "$n" -le 3 ]; do
    echo "tour $n"
    n=$((n + 1))
done

while read -r ligne; do
    echo "lu : $ligne"
done < fichier.txt
```

- `break` quitte la boucle / `continue` passe au tour suivant

## Fonctions

```bash
sauvegarde() {
    tar -czf "/backup/$1-$(date +%F).tar.gz" "$1"
}
sauvegarde /var/www
```

## Sécuriser un script

- `set -e` → arrête le script à la première commande qui échoue
- `set -u` → erreur si une variable n'est pas définie ; `set -o pipefail` → un pipe échoue si UNE de ses commandes échoue ; souvent combinés : `set -euo pipefail`
- `command -v nginx > /dev/null || { echo "nginx absent" >&2; exit 1; }` → vérifie qu'un programme existe
- `tmp=$(mktemp)` → crée un fichier temporaire unique ; `mktemp -d` → un dossier temporaire

## Exemple complet : sauvegarde

```bash
#!/bin/bash
set -euo pipefail

src=${1:?usage : $0 dossier}
dest=/backup/$(basename "$src")-$(date +%F).tar.gz

tar -czf "$dest" "$src"
echo "Sauvegarde OK : $dest"
```

À planifier avec cron ou un timer (voir [cron / at](cron-at.md)).
