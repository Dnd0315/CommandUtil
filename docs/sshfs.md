# SSHFS (monter un dossier distant via SSH)

[← Index](../README.md)

- `sudo apt install sshfs` → installation
- `sudo mkdir -p /app-srv1/data-export` → crée le point de montage local (d'abord !)
- `sudo sshfs -o allow_other,rw app-srv1:/remote-data-export /app-srv1/data-export` → monte le dossier /data-export du serveur app-srv1 en lecture-écriture ; allow_other = les autres utilisateurs (pas seulement celui qui monte) y accèdent. Format : `serveur:/chemin_distant /point_de_montage_local`
- Vérifier : `find /app-srv1/data-export` ; `touch /app-srv1/data-export/new` puis `ssh app-srv1 find /data-export` (le fichier apparaît des deux côtés)
- `man sshfs` → options
- `sudo umount /app-srv1/data-export` (ou `fusermount -u`) → démonte
