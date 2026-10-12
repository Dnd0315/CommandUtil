# Boot, processus

[← Index](../README.md)

## Arrêt, redémarrage, mode de démarrage

- `sudo systemctl reboot` → redémarre ; `--force` si un service bloque le redémarrage
- `sudo systemctl poweroff` → éteint la machine
- `sudo shutdown 02:00` / `sudo shutdown +15` → extinction programmée à 2h / dans 15 min
- `sudo shutdown -r 02:00` → redémarrage programmé (-r = reboot)
- `sudo shutdown -c` → annule une extinction programmée
- `systemctl get-default` → affiche la cible de démarrage (graphical ou multi-user)
- `sudo systemctl set-default multi-user.target` → démarre désormais en mode texte (graphical.target = interface graphique)
- `sudo systemctl isolate multi-user.target` → bascule IMMÉDIATEMENT vers cette cible, sans redémarrer (`isolate graphical.target` pour revenir à l'interface graphique). Modes de secours (rescue, emergency) et GRUB : voir [GRUB et récupération](grub-recuperation.md)

## Priorité (niceness)

Priorité (niceness) = indique à Linux quels programmes passent EN PREMIER quand le processeur est saturé. « nice » = gentil : un processus à niceness haute laisse passer les autres.

- -20 = le plus prioritaire (égoïste) / 0 = normal (valeur par défaut) / 19 = le moins prioritaire (gentil). ⚠️ Plus le chiffre est BAS, plus c'est prioritaire
- Un utilisateur normal peut seulement monter vers 19 (se rendre plus gentil) ; descendre vers -20 demande sudo (sinon tout le monde se mettrait à -20)
- N'a d'effet que si le processeur est saturé : sur une machine inactive, tout tourne à pleine vitesse
- `nice -n 19 tar -czf /backup/site.tar.gz /var/www` → sauvegarde qui ne ralentit pas le serveur web
- `nice -n 11 bash` → LANCE bash avec une niceness de 11
- `renice -n 7 -p 1234` → CHANGE la niceness d'un processus déjà en cours (PID 1234)
- `ps -o pid,ni,cmd -p 1234` → affiche la niceness d'un processus (colonne NI, aussi visible dans top)

## Lister les processus

- `ps aux` / `ps -ef` → liste tous les processus (PID, utilisateur, CPU, commande)
- `ps aux --sort=-%mem | head` → processus triés par mémoire (le plus gros en haut)
- `top` → vue en direct des processus (q pour quitter)
- `top -b -n 1 | grep nom` → une capture de top ; colonnes VSZ = mémoire virtuelle, RSS = mémoire réellement utilisée
- `pgrep -a bash` → trouve les PID des processus nommés bash (-a = affiche la commande complète)
- `lsof -p 1234` → liste les fichiers ouverts par le processus 1234
- `lsof /path` → montre quel processus utilise ce fichier ou dossier (utile si umount répond « busy »)

## Signaux

Signaux = messages courts que Linux envoie à un processus (s'arrêter, se mettre en pause, recharger sa config…). kill ne « tue » pas forcément : il ENVOIE UN SIGNAL (15 par défaut).

- 15 SIGTERM → « arrête-toi proprement » : le processus peut finir et sauvegarder. Signal par défaut de kill
- 9 SIGKILL → « arrête-toi immédiatement » : impossible à refuser, aucune sauvegarde
- 1 SIGHUP → souvent utilisé pour « recharge ta configuration » sans s'arrêter
- 2 SIGINT → interruption = ce qu'envoie Ctrl+C
- 19 SIGSTOP → met en pause (comme Ctrl+Z) / 18 SIGCONT → relance un processus en pause

Commandes :

- `kill 1234` → envoie SIGTERM (15) : à essayer EN PREMIER
- `kill -9 1234` = `kill -KILL 1234` = `kill -SIGKILL 1234` → arrêt forcé, seulement si SIGTERM ne suffit pas
- `kill -HUP 1234` → demande au processus de recharger sa config
- `kill -l` → affiche la LISTE de tous les signaux avec leur numéro (-l = list) ; n'envoie rien, c'est un aide-mémoire
- `pkill nginx` → envoie le signal aux processus par NOM plutôt que par PID

## Premier plan / arrière-plan

- `commande &` → lance en arrière-plan
- `Ctrl+Z` → met en pause le processus au premier plan
- `jobs` → liste les tâches du terminal ; `fg %1` la ramène au premier plan ; `bg %1` la relance en arrière-plan

## Appels système (strace)

- `sudo strace -p 3612` → affiche en direct les appels système (syscalls) du processus 3612 ; laisser tourner quelques secondes (Ctrl+C pour arrêter) et chercher la ligne voulue, ex : kill(666, SIGTERM)
- `sudo strace -p 3612 -e trace=kill` → n'affiche que les appels kill ; `-f` suit aussi les processus enfants
