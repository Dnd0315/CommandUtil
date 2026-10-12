# systemd (services)

[← Index](../README.md)

## Gérer un service

- `systemctl status nginx` → état du service (actif, activé au boot, derniers logs)
- `sudo systemctl start / stop / restart nginx` → démarre / arrête / redémarre maintenant
- `sudo systemctl reload nginx` → recharge la config sans couper le service
- `sudo systemctl enable nginx` → démarrage automatique au boot (ne le lance PAS maintenant)
- `sudo systemctl enable --now nginx` → activé au boot ET lancé tout de suite (le plus courant à l'examen)
- `sudo systemctl disable --now nginx` → désactivé au boot et arrêté
- `sudo systemctl mask nginx` → bloque totalement le service (impossible à démarrer, même manuellement) ; `unmask` pour annuler

Pour les logs d'un service : voir [Logs](logs.md) (`journalctl -u`, `journalctl -xeu`).

## Diagnostiquer

- `systemctl is-active nginx` / `systemctl is-enabled nginx` / `systemctl is-failed nginx` → réponse en un mot (et code de retour utilisable dans un script)
- `systemctl --failed` → toutes les unités en échec ; `sudo systemctl reset-failed` → efface cet état d'échec
- `systemctl list-unit-files --type=service` → tous les services installés et leur état (enabled, disabled, masked, static)
- `systemctl show nginx -p ExecStart` → une propriété précise de l'unité (sans `-p` : toutes)
- `systemd-analyze blame` → temps de démarrage de chaque unité (la plus lente en premier) / `systemd-analyze critical-chain` → chaîne qui retarde le démarrage
- `systemd-analyze verify /etc/systemd/system/x.service` → vérifie la syntaxe d'un fichier d'unité

## Modifier un service

- `systemctl cat nginx` → affiche le fichier d'unité du service
- `sudo systemctl edit nginx` → modifie un service proprement : crée un « override » sans toucher au fichier d'origine (/etc/systemd/system/nginx.service.d/override.conf)
- `sudo systemctl edit --full nginx` → édite le fichier d'unité complet
- `sudo systemctl daemon-reload` → recharge la configuration de systemd (fichiers d'unité) après une modification

Dans un override, pour REMPLACER une directive à valeurs multiples comme `ExecStart=`, la vider d'abord :

```ini
[Service]
Environment=MODE=prod
ExecStart=
ExecStart=/usr/local/bin/app --port 8080
```

## Créer un service

Fichier `/etc/systemd/system/app.service`, puis `daemon-reload`, puis `enable --now` :

```ini
[Unit]
Description=Mon application
After=network.target

[Service]
ExecStart=/usr/local/bin/app
User=app
WorkingDirectory=/opt/app
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Clés utiles :

- `[Unit]` : `Description=` / `After=` (ordre de démarrage) / `Requires=` ou `Wants=` (dépendance forte / faible)
- `[Service]` : `ExecStart=` (commande) / `ExecStartPre=` / `ExecStop=` / `ExecReload=` / `User=` / `WorkingDirectory=` / `Environment=` / `Restart=always` (ou `on-failure`) / `Type=` (`simple` par défaut, `forking` si le programme se détache, `oneshot` pour une tâche qui se termine)
- Limites de ressources : `MemoryMax=500M` / `CPUQuota=50%` / `LimitNOFILE=4096`
- `[Install]` : `WantedBy=multi-user.target` (condition pour que `enable` fonctionne)

Tâches planifiées avec un fichier .timer : voir [cron / at](cron-at.md).

## Lister

- `systemctl list-units --type=service` → liste les services chargés
- `systemctl list-dependencies` → arbre des dépendances entre unités

## À ne pas confondre

⚠️ Ordre des mots : `systemctl ACTION NOM` (systemctl restart nginx) mais `service NOM ACTION` (service nginx restart). Écrire « service restart nginx » donne l'erreur « restart: unrecognized service ».
