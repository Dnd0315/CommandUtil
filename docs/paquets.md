# Paquets (apt / dpkg)

[← Index](../README.md)

Pour RHEL / Fedora (dnf, rpm) : voir [Équivalents RHEL / Fedora](rhel-fedora.md).

## apt

- `sudo apt update` → met à jour la LISTE des paquets disponibles (n'installe rien)
- `sudo apt upgrade` → installe les mises à jour des paquets déjà installés (sans en supprimer) ; `sudo apt full-upgrade` → idem, mais peut installer / supprimer des paquets pour résoudre les dépendances (changement de noyau…)
- `apt search nginx` → cherche un paquet / `apt show nginx` → détails d'un paquet
- `apt list --installed` → paquets installés ; `apt list --upgradable` → mises à jour disponibles
- `apt policy nginx` (ou `apt-cache policy nginx`) → version installée, version candidate et dépôt d'origine
- `sudo apt install nginx` → installe le paquet
- `sudo apt install ./paquet.deb` → installe un .deb local EN résolvant les dépendances
- `sudo apt remove nginx` → désinstalle le programme mais GARDE ses fichiers de configuration (utile si on le réinstalle plus tard)
- `sudo apt purge nginx` → désinstalle le programme ET supprime sa configuration (désinstallation complète)
- `sudo apt autoremove` → supprime les dépendances devenues inutiles
- `sudo apt clean` → vide le cache des paquets téléchargés (/var/cache/apt/archives)
- `sudo apt-mark hold nginx` → fige le paquet (plus de mise à jour) ; `apt-mark unhold nginx` → libère ; `apt-mark showhold` → liste les paquets figés

## dpkg

- `dpkg -l` → liste les paquets installés (`dpkg -l | grep nginx` pour filtrer)
- `dpkg -s nginx` → statut et informations d'un paquet installé
- `dpkg --listfiles nginx` → liste les fichiers installés par un paquet
- `dpkg -S /usr/bin/ls` → indique quel paquet a installé ce fichier
- `sudo dpkg -i paquet.deb` → installe un .deb SANS résoudre les dépendances ; en cas d'erreur de dépendances : `sudo apt -f install` termine l'installation
- `sudo dpkg -r nginx` → supprime (garde la config) / `sudo dpkg -P nginx` → purge
- `sudo dpkg-reconfigure tzdata` → relance la configuration interactive d'un paquet

## Dépôts

- `/etc/apt/sources.list` et `/etc/apt/sources.list.d/` → dépôts utilisés par apt
- `sudo add-apt-repository ppa:utilisateur/depot` → ajoute un PPA Ubuntu (paquet software-properties-common), puis `sudo apt update`
- Dépôt tiers à la main : récupérer la clé, puis déclarer le dépôt dans un fichier .list :

  ```bash
  curl -fsSL https://repo.example.com/key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/example.gpg
  echo "deb [signed-by=/usr/share/keyrings/example.gpg] https://repo.example.com/ubuntu jammy main" | sudo tee /etc/apt/sources.list.d/example.list
  ```
