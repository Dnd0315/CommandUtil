# Git

[← Index](../README.md)

## Bases

- `git clone url` → copie un dépôt distant ; dépôt LOCAL (chemin) vers un dossier précis : `git clone /repositories/auto-verifier ~/repositories/auto-verifier`, puis `cd` dedans
- `git init` → crée un dépôt dans le dossier courant
- `git add file` → prépare un fichier pour le prochain commit (`git add .` = tout)
- `git commit -m "message"` → enregistre les changements préparés
- `git status` / `git log` → état actuel / historique des commits
- `git remote -v` → dépôts distants configurés
- `git push` / `git pull` → envoie / récupère les commits du dépôt distant

## Branches

- `git branch dev` → crée une branche ; `git checkout -b dev` → la crée et bascule dessus
- `git branch -a` → liste TOUTES les branches (locales + distantes remotes/origin/…)
- `git checkout dev5` → bascule sur la branche (crée la locale à partir de origin/dev5)
- `git merge dev` → fusionne dev dans la branche courante. Exemple : `git checkout main` puis `git merge dev5` → fusionne UNIQUEMENT dev5 dans main (« Fast-forward » = rien à fusionner, juste avancé). Vérifier : grep dans le fichier ; `git log`

## Trouver quelle branche contient un contenu

- `git checkout dev4` puis `grep user_registration_level config.yaml` (répéter sur chaque branche)
- Sans changer de branche : `git diff main origin/dev5` / `git diff origin/dev5` → montre les différences ; `git log` → les commits

## Dossier vide

Un dossier VIDE n'est pas suivi par git : `mkdir logs ; touch logs/.keep` (fichier caché vide) puis `git add logs` puis `git commit -m 'added log directory'` → le dossier est commité. Vérifier avant avec `git status` (« Changes to be committed : new file: logs/.keep »).
