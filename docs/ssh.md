# SSH

[← Index](../README.md)

## Clés

Pour comprendre les paires de clés (privée / publique) : voir [SSL / TLS](ssl-tls.md).

- `ssh-keygen -t ed25519 -C "your_email@example.com"` → génère une paire de clés dans ~/.ssh/ (id_ed25519 = privée, .pub = publique)
- `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"` → alternative en RSA 4096 bits
- `ssh-keygen -lf ~/.ssh/id_ed25519.pub` → empreinte (fingerprint) d'une clé ; `ssh-keygen -p -f ~/.ssh/id_ed25519` → change sa passphrase
- `ssh-copy-id john@host` → ajoute ta clé publique à une machine (la copie dans ~/.ssh/authorized_keys du serveur)
- `~/.ssh/authorized_keys` → clés autorisées (chmod 600 ; dossier ~/.ssh en chmod 700) ; `cat ~/.ssh/authorized_keys` → voir les clés déjà autorisées sur une machine

## Client

- `ssh john@host` → se connecte ; `ssh -p 2222 john@host` → autre port ; `ssh -i ~/.ssh/autre_cle john@host` → autre clé privée
- `ssh john@host 'df -h'` → exécute UNE commande à distance puis revient
- `ssh -v john@host` → mode débogage (`-vvv` plus bavard) : voir pourquoi une clé est refusée
- `ssh -J bastion john@cible` → passe par un serveur de rebond (jump host)
- `~/.ssh/config` → config client (alias d'hôtes, user, port) ; chmod 600. Exemple, puis `ssh web1` suffit :

  ```
  Host web1
      HostName 10.0.0.10
      User john
      Port 2222
      IdentityFile ~/.ssh/id_ed25519
  ```

- `ssh-keygen -R host` → supprime l'hôte de known_hosts (message « REMOTE HOST IDENTIFICATION HAS CHANGED » après réinstallation du serveur)
- `ssh-keyscan host >> ~/.ssh/known_hosts` → enregistre la clé publique d'un serveur à l'avance
- `eval "$(ssh-agent)"` puis `ssh-add ~/.ssh/id_ed25519` → saisir la passphrase une seule fois pour la session ; `ssh-add -l` → liste les clés chargées

## Copier des fichiers

- `scp file john@host:/tmp/` → copie vers le serveur / `scp john@host:/etc/hosts .` → depuis le serveur / `scp -r dir john@host:/tmp/` → un dossier
- `scp -P 2222 file john@host:/tmp/` → autre port (⚠️ P MAJUSCULE, contrairement à `ssh -p`)
- `sftp john@host` → session interactive : `ls`, `cd`, `get fichier`, `put fichier`, `lcd` (dossier local), `exit`
- Synchronisation de dossiers par SSH : `rsync` (voir [Archives](archives.md))

## Tunnels (redirections de port)

- `ssh -L 8080:localhost:80 john@host` → redirection LOCALE : http://localhost:8080 sur MA machine atteint le port 80 vu depuis host
- `ssh -R 9000:localhost:22 john@host` → redirection INVERSE : le port 9000 de host revient vers le port 22 de MA machine
- `ssh -D 1080 john@host` → proxy SOCKS sur localhost:1080 (le trafic du navigateur sort par host)
- `-N` (pas de shell) et `-f` (arrière-plan) se combinent avec les trois : `ssh -fN -L 8080:localhost:80 john@host`

## Serveur

- `/etc/ssh/sshd_config` → config du serveur SSH : `PermitRootLogin no` (interdit root), `PasswordAuthentication no` (clés uniquement), `X11Forwarding yes` (applis graphiques à distance), `Port 2222`
- Restrictions d'accès : `AllowUsers john jane` / `AllowGroups sshusers` (liste blanche) / `DenyUsers bob` / `MaxAuthTries 3` (essais par connexion) / `ClientAliveInterval 300` avec `ClientAliveCountMax 2` (coupe les sessions inactives) / `ListenAddress 10.0.0.5` (écoute sur cette IP seulement) / `Banner /etc/issue.net` (message avant la connexion)
- Bloc `Match` : réglages valables pour certains utilisateurs / réseaux, jusqu'au prochain Match. Exemple : `Match Address 10.0.0.0/24` suivi (indenté) de `PasswordAuthentication yes` → mot de passe autorisé seulement depuis ce réseau
- `sudo sshd -t` → teste la syntaxe de la config AVANT de redémarrer (silence = OK)
- `sudo systemctl restart ssh` → obligatoire pour appliquer les changements (RHEL : service `sshd`)
- `sudo sshd -T` → affiche la liste complète de ce que le serveur applique réellement
- ⚠️ Changer le port : autoriser le nouveau port dans le pare-feu et, sous SELinux, déclarer le port (`semanage port`, voir [SELinux / AppArmor](selinux-apparmor.md)). Sur Ubuntu 24.04, sshd est lancé par un socket (`ssh.socket`) qui ignore `Port` de sshd_config : soit désactiver le socket (`sudo systemctl disable --now ssh.socket` puis `sudo systemctl enable --now ssh`), soit mettre `ListenStream=2222` dans un override de ssh.socket (`systemctl edit ssh.socket`, section `[Socket]`, en vidant d'abord `ListenStream=`)
