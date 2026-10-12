# VM / conteneurs

[← Index](../README.md)

## VM (QEMU-KVM / virsh)

QEMU-KVM : virtualisation intégrée au noyau Linux ; virsh : la gérer en ligne de commande (libvirt).

### Prérequis

- `grep -Ec '(vmx|svm)' /proc/cpuinfo` → supérieur à 0 = le processeur gère la virtualisation matérielle
- `virt-host-validate` → vérifie que l'hôte est prêt pour KVM / libvirt (chaque ligne doit être PASS)
- `sudo apt install qemu-kvm libvirt-daemon-system virtinst` → installation ; activer ensuite le service libvirtd (voir [systemd](systemd.md)) et ajouter l'utilisateur au groupe libvirt pour gérer les VM sans sudo (voir [Utilisateurs](utilisateurs.md))

### Gérer

- `virsh list --all` → toutes les VM, y compris éteintes
- `virsh start vm1` → démarre / `virsh shutdown vm1` → arrêt propre / `virsh destroy vm1` → arrêt forcé (ne supprime PAS la VM)
- `virsh reboot vm1` → redémarre la VM
- `virsh suspend vm1` / `virsh resume vm1` → met en pause / reprend
- `virsh autostart vm1` → démarre la VM avec l'hôte (--disable pour annuler)
- `virsh dominfo vm1` → infos : CPU, RAM, état, autostart
- `virsh console vm1` → console série de la VM (Ctrl+] pour sortir)
- `virsh domifaddr vm1` → adresse IP de la VM
- `virsh domblklist vm1` → disques attachés à la VM
- `virsh setvcpus vm1 2 --config --maximum` puis `virsh setvcpus vm1 2 --config` → change le nombre de CPU
- `virsh setmaxmem vm1 2G --config` puis `virsh setmem vm1 2G --config` → change la RAM
- `virsh edit vm1` → édite la définition XML de la VM
- `virsh dumpxml vm1 > vm1.xml` → exporte la définition ; `virsh define vm1.xml` → (re)déclare une VM à partir d'un XML
- `virsh undefine vm1` → supprime la définition de la VM (`--remove-all-storage` supprime aussi ses disques)

### Créer et cloner

- `virt-install --name vm1 --memory 1024 --vcpus 1 --disk size=10 --cdrom image.iso` → crée une VM en l'installant depuis un ISO
- `virt-install --name vm1 --memory 1024 --vcpus 1 --disk path=/var/lib/libvirt/images/ubuntu.img --import --osinfo ubuntu22.04 --noautoconsole` → crée une VM à partir d'un disque EXISTANT. ⚠️ --import est un simple drapeau SANS valeur : le chemin du disque va dans `--disk path=…` (sinon erreur « unrecognized arguments ») ; --noautoconsole = ne pas rester bloqué dans la console
- `virt-install --osinfo list` → noms d'OS valides
- `--network bridge=br0` (option de virt-install) → branche la VM sur un bridge de l'hôte (voir [Réseau](reseau.md)) ; par défaut, réseau NAT « default »
- `virt-clone --original vm1 --name vm2 --auto-clone` → clone une VM éteinte (nouveau disque, nouvelle adresse MAC)

### Disques (qemu-img) et snapshots

- `qemu-img create -f qcow2 /var/lib/libvirt/images/extra.qcow2 5G` → crée un disque virtuel de 5 Go au format qcow2 (alloué au fur et à mesure)
- `qemu-img info disque.qcow2` → format, taille virtuelle, taille réelle, snapshots
- `qemu-img resize disque.qcow2 +5G` → agrandit le disque (VM éteinte) ; agrandir ensuite partition et système de fichiers DANS la VM
- `qemu-img convert -f raw -O qcow2 disque.img disque.qcow2` → convertit de format
- `virsh attach-disk vm1 /var/lib/libvirt/images/extra.qcow2 vdb --subdriver qcow2 --persistent` → ajoute un disque à la VM (visible comme /dev/vdb) ; `virsh detach-disk vm1 vdb --persistent` → le retire
- `virsh snapshot-create-as vm1 snap1` → crée un snapshot ; `virsh snapshot-list vm1` → les liste
- `virsh snapshot-revert vm1 snap1` → revient à ce snapshot ; `virsh snapshot-delete vm1 snap1` → le supprime

### Réseau libvirt

- `virsh net-list --all` → réseaux virtuels (le réseau « default » est un NAT en 192.168.122.0/24 via virbr0)
- `virsh net-start default` → démarre un réseau / `virsh net-autostart default` → le démarre avec l'hôte

## Docker

### Images

- `docker pull nginx:alpine` → télécharge une image sans la lancer / `docker search nginx` → cherche sur le registre
- `docker images` → images locales / `docker rmi nginx` → supprime une image
- `docker tag nginx:alpine monrepo/web:v1` → donne un autre nom / une autre version à une image ; `docker login` puis `docker push monrepo/web:v1` → publie
- `docker save -o web.tar web:v1` → exporte une image dans un fichier / `docker load -i web.tar` → l'importe (transfert sans registre)
- `docker commit web web:v2` → crée une image à partir de l'état d'un conteneur

### Construire une image

Fichier `Dockerfile` (ou `Containerfile`) dans un dossier :

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

- `docker build -t web:v1 .` → construit l'image web:v1 avec le Dockerfile du dossier courant (le `.` = contexte de build) ; `-f autre_fichier` pour un autre nom de fichier
- Instructions : `FROM` image de base / `RUN` commande exécutée pendant la construction / `COPY` copie des fichiers dans l'image / `WORKDIR` dossier de travail / `ENV` variable d'environnement / `EXPOSE` port documenté / `USER` utilisateur / `CMD` commande par défaut (remplaçable à `docker run`) / `ENTRYPOINT` commande toujours exécutée

### Lancer

- `docker run -d -p 8080:80 --name web --restart always nginx` → lance un conteneur nginx en arrière-plan (-d), port 8080 de l'hôte → 80 du conteneur, redémarre automatiquement
- `docker run -d --name frontend_v3 --memory 30m -p 1234:80 nginx:alpine` → conteneur détaché (-d), limite mémoire 30 Mo (--memory 30m), port hôte 1234 → port conteneur 80. L'IMAGE vient en dernier, les options AVANT l'image
- `docker run -v /hote:/conteneur` → monte un dossier (bind mount) ; ajouter `:ro` pour la lecture seule
- Autres options : `-e VAR=valeur` (variable d'environnement) / `--env-file fichier` / `-it` (interactif + terminal, ex : `docker run -it ubuntu bash`) / `--rm` (supprime le conteneur à son arrêt) / `--cpus 0.5` / `-u 1000` (utilisateur) / `-w /app` (dossier de travail) / `--read-only`
- `docker run --help` → liste les options (--cpus, --network, -v, --pids-limit…)

### Gérer

- `docker ps` → conteneurs en cours / `docker ps -a` → tous, y compris arrêtés
- `docker start web` / `docker restart web` → relance un conteneur arrêté / le redémarre
- `docker stop web` → arrête (le conteneur reste visible avec `docker ps -a`)
- `docker rm web` → supprime le conteneur ; `docker rm --force web` → supprime même s'il tourne
- `docker logs web` → sortie du conteneur ; `docker logs -f --tail 50 web` → suit les 50 dernières lignes en direct
- `docker exec -it web bash` → ouvre un shell dans le conteneur
- `docker cp web:/etc/nginx/nginx.conf .` → copie un fichier du conteneur vers l'hôte (et inversement)
- `docker port web` → ports publiés
- `docker top web` → processus du conteneur / `docker stats --no-stream` → consommation CPU / mémoire
- `docker update --restart unless-stopped web` → change la politique de redémarrage d'un conteneur existant (`--memory 50m` pour la mémoire)
- `docker system df` → espace utilisé par images, conteneurs et volumes / `docker system prune` → supprime conteneurs arrêtés, images et réseaux inutilisés

### Volumes et réseaux

- `docker volume create data` → crée un volume géré par Docker (stocké sous /var/lib/docker/volumes/) / `docker volume ls` / `docker volume inspect data` / `docker volume rm data`
- `docker run -v data:/var/lib/app …` → monte un volume NOMMÉ (survit à la suppression du conteneur ; sans / devant le nom = volume, avec / = bind mount)
- `docker network create mynet` → crée un réseau / `docker network ls` / `docker network inspect mynet`
- `docker run --network mynet --name db …` → sur un réseau créé par l'utilisateur, les conteneurs se joignent par leur NOM (DNS intégré)
- `docker network connect mynet web` → ajoute un conteneur existant à un réseau

### Inspecter

- `sudo docker inspect frontend_v2` → toute la configuration en JSON (`| vim -` pour chercher). IP du conteneur : champ NetworkSettings → Networks → IPAddress ; volume monté : champ Mounts → Destination (dans le conteneur) / Source (sur l'hôte)
- `docker inspect -f '{{.NetworkSettings.IPAddress}}' frontend_v2` → affiche juste l'IP
- `docker inspect -f '{{range .Mounts}}{{.Destination}}{{end}}' frontend_v2` → juste le dossier de montage

### Service Docker

- `docker info` → configuration du démon (pilote de stockage, racine /var/lib/docker, nombre de conteneurs) / `docker version` → versions client et serveur
- `/etc/docker/daemon.json` → configuration du démon (ex : `{"log-driver": "json-file"}`) ; relancer le service docker après modification
- Utiliser docker sans sudo : ajouter l'utilisateur au groupe docker (voir [Utilisateurs](utilisateurs.md))

## Podman

Même syntaxe que Docker : remplacer `docker` par `podman` (run, ps, images, pull, build, exec, logs, inspect, volume…). Différences :

- Pas de démon, et rootless par défaut (un utilisateur normal lance ses conteneurs sans sudo)
- Utiliser les noms complets d'image : `podman pull docker.io/library/nginx` (registres de recherche dans `/etc/containers/registries.conf`)
- `loginctl enable-linger john` → les conteneurs rootless de john continuent de tourner après sa déconnexion
- `podman generate systemd --new --name web` → génère une unité systemd pour un conteneur (ancienne méthode, remplacée par Quadlet)
- Quadlet : fichier `/etc/containers/systemd/web.container`, puis recharger systemd (voir [systemd](systemd.md)) → le service `web.service` est créé automatiquement :

  ```
  [Container]
  Image=docker.io/library/nginx:alpine
  PublishPort=8080:80

  [Service]
  Restart=always

  [Install]
  WantedBy=multi-user.target
  ```

## Dépannage

Si `docker run` échoue avec une erreur iptables : supprimer le conteneur (`docker rm --force nom`), `sudo service docker restart`, relancer. Test : `curl localhost:1234`
