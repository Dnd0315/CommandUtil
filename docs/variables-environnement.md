# Variables d'environnement

[← Index](../README.md)

## Définir

- `VAR=valeur` → variable locale : visible UNIQUEMENT dans le shell/script où elle est définie
- `export VAR=valeur` → variable d'environnement : héritée par tous les processus enfants (bash lancé dedans, scripts…). Exemple : `export NOM_DE_LA_VARIABLE="sa_valeur"`

Démonstration : seule la variable exportée passe au sous-shell.

```bash
TEST1=a
export TEST2=b
bash
echo $TEST1   # vide
echo $TEST2   # b
```

## Consulter

- `echo $VAR` → affiche la valeur
- `env` (ou `printenv`) → liste les variables exportées ; `env | grep VAR` → filtre
- `set` → TOUTES les variables du shell (locales et exportées) et les fonctions
- `unset VAR` → supprime
- `cat ~/.bashrc | grep VARIABLE1` → vérifie où une variable est définie. Ne PAS modifier .bashrc si la consigne l'interdit : tout se fait dans le script

## PATH et alias

- `echo $PATH` → dossiers où le shell cherche les commandes (séparés par :)
- `export PATH=$PATH:/opt/app/bin` → ajoute un dossier À LA FIN du PATH (au début : `/opt/app/bin:$PATH`) ; à mettre dans un fichier de profil pour le garder
- `alias ll='ls -lah'` → crée un raccourci (valable dans le shell courant) ; `alias` seul → liste les alias ; `unalias ll` → le supprime

## Sauvegarder sur la machine

Ajouter la ligne au fichier de configuration du shell :

```bash
# Pour Bash
echo 'export NOM_DE_LA_VARIABLE="sa_valeur"' >> ~/.bashrc

# Pour Zsh
echo 'export NOM_DE_LA_VARIABLE="sa_valeur"' >> ~/.zshrc
```

Appliquer les modifications :

```bash
# Pour Bash
source ~/.bashrc

# Pour Zsh
source ~/.zshrc
```

## Quel fichier est lu et quand

- Shell de LOGIN (connexion console, SSH, `su - john`, `bash -l`) : lit `/etc/profile` (et les scripts `/etc/profile.d/*.sh`), puis le PREMIER trouvé parmi `~/.bash_profile`, `~/.bash_login`, `~/.profile`
- Shell interactif NON login (terminal ouvert dans l'interface graphique, `bash` lancé dans un shell, `su john`) : lit `/etc/bash.bashrc` (RHEL : `/etc/bashrc`) puis `~/.bashrc`
- `~/.bash_profile` et `~/.profile` chargent en général `~/.bashrc` : une variable mise dans ~/.bashrc est donc vue dans les deux cas
- `echo $0` → `-bash` (avec le tiret) = shell de login
- `~/.bash_logout` → exécuté à la déconnexion d'un shell de login

Pour TOUS les utilisateurs :

- `/etc/profile.d/mon-app.sh` → script lu par /etc/profile à chaque connexion (le plus propre pour définir une variable ou ajouter au PATH pour tout le monde)
- `/etc/environment` → fichier simple de lignes `VAR=valeur` (sans `export`, sans commandes, pas d'expansion), lu par PAM à l'ouverture de session
- `/etc/skel/` → modèle copié dans le home des FUTURS comptes (voir [Utilisateurs](utilisateurs.md))
