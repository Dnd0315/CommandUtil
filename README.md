# Liste des commandes utiles pour ingénieur IT

## Administration

### Securité

Asymmetric encryption uses a Pair of Key: private key and publick key

Pour generer les paires de clé:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
ou

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
et avec openssl

```bash
openssl genrsa -out myserver.key 2048
```

puis 

```bash
openssl rsa -in myserver.key -pubout > myserver.pen 
```

Certificate Authority (Symantec, Digicert, Comodo, GlobalSign)

Generer un Certificate Signing Request (CSR) 

```bash
openssl req -new -key myserver.key -out myserver.csr -subj "/C=UAE/ST=DU/O=TLT, Inc./CN=my-server.com"


Ajouter une clé publique à une machine:

```bash
cat ~/.ssh/authorized_keys
```

### Réseau


Vérifier si un service écoute sur un port en particulier:

```bash
sudo lsof -i -P -n | grep LISTEN | grep 8080
```


### Système 

Créer une variable d'environnement:

```bash
export NOM_DE_LA_VARIABLE="sa_valeur"
```

Sauvergarder une variable sur la machine:

```bash
# Pour Bash
echo 'export NOM_DE_LA_VARIABLE="sa_valeur"' >> ~/.bashrc

# Pour Zsh
echo 'export NOM_DE_LA_VARIABLE="sa_valeur"' >> ~/.zshrc
```

Appliquer les modifications:

```bash
# Pour Bash
source ~/.bashrc

# Pour Zsh
source ~/.zshrc
````


#### Service

Vérifier l'état d'un service:

```bash
sudo systemctl status app.service
```

Vérifier les logs d'un service:

```bash
sudo journalctl -xeu app.service
```

Modifier un service proprement:

```bash
sudo systemctl edit app
```

Recharger la configuration de `systemd`:

```bash
sudo systemctl daemon-reload
```

Redémarer le service: 

```bash
sudo systemctl restart app
```



