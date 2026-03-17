
# Introduction
## Contexte général 

**Docker** est une plateforme de conteneurisation open source qui permet aux équipes de développement d'empaqueter leurs applications et toutes leurs dépendances dans des conteneurs légers et portables.

En gros Docker c'est la solution au problème informatique le plus courant

>*mais pourtant ça marche sur mon ordi*

Docker va permettre d’exécuter un service donné, dans un environnement donné, et donc aucun paramètre lié à votre machine ou votre contexte ne va faire varier le résultat.

## Cas d'usage en self-hosting

Dans le cas du homelab, utiliser docker nous sert surtout à compartimenter chacun de nos services dans leurs petites boîtes noires respectives.

Chaque service a son environnement, sa configuration réseau etc.

# Installation

https://docs.docker.com/engine/install/debian/

Pour l'installer, il suffit de suivre les différentes étapes du lien ci dessus, que je vais reprendre ici étape par étape :

*Ajout du dépôt docker à votre gestionnaire de paquet (Apt)*
```bash
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

*Installation des outils docker via Apt*
```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```


## Docker vs Docker Compose

### Docker

Docker est un outil qui permet de lancer **des conteneurs**.  
Un conteneur est une application avec tout ce dont elle a besoin pour fonctionner.

Avec Docker “simple”, on lance chaque conteneur **avec une commande**.

Exemple :

```bash
docker run -d \  
  -p 8080:80 \  
  --name mon-nginx \  
  nginx
```

Ici on démarre un serveur web avec l’image **nginx**.

Le problème est que les commandes peuvent devenir **longues et difficiles à maintenir**, surtout quand une application a plusieurs services.

---
### Docker Compose

Docker Compose permet de **décrire plusieurs conteneurs dans un fichier** (`compose.yml`).

Au lieu d’une longue commande, on écrit une configuration.

Exemple :

```yaml
services:  
  web:  
    image: nginx  
    ports:  
      - "8080:80"
```

Puis on lance tout simplement :

```bash
docker compose up -d
```

---

### Différence principale

|Docker|Docker Compose|
|---|---|
|Lance des conteneurs **un par un**|Lance **plusieurs conteneurs ensemble**|
|Configuration dans la **ligne de commande**|Configuration dans un **fichier YAML**|
|Peu pratique pour les applications complexes|Idéal pour les stacks complètes|

---

### Exemple concret en self-hosting

Beaucoup d’applications auto-hébergées nécessitent **plusieurs services** :

- l’application

- une base de données

- parfois un cache (Redis)

- parfois un reverse proxy


Avec Docker seul, il faudrait lancer **chaque conteneur séparément**.

Avec Docker Compose, tout est défini dans **un seul fichier**, ce qui permet :

- de démarrer la stack facilement

- de la sauvegarder dans Git

- de la partager.


---
En résumé,

> **Docker sert à lancer des conteneurs. Docker Compose sert à organiser et lancer plusieurs conteneurs ensemble avec un fichier de configuration.**

# Utilisation

Vous pouvez normalement désormais déclarer votre premier docker compose.

En suivant la trame [Procédure d'installation d'un nouveau service](Procédure%20d'installation%20d'un%20nouveau%20service.md)

## Utiliser Docker sans sudo

Après l'installation, les commandes Docker doivent généralement être exécutées avec `sudo` :  
  
```bash  
sudo docker ps
```

Cela s'explique parce que le service Docker s'exécute avec des privilèges élevés et que l'accès au socket Docker est limité aux utilisateurs autorisés.

Pour pouvoir utiliser Docker **sans `sudo`**, il suffit d'ajouter votre utilisateur au groupe `docker` avec les commandes suivantes :

```bash
sudo groupadd docker
sudo gpasswd -a $USER docker
```


