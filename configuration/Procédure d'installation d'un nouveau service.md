# Sommaire
- Récupération et modification du compose.yaml
- Lancement et test du service
- Diagnostic et debugging
-  Intégration du service dans le tunnel cloudflare
# Récupération et modification du compose.yml

Afin de créer un nouveau service, on va taper `<NOM DU SERVICE> docker compose` sur les internets afin de trouver un exemple de fichier `compose.yml` et pouvoir le paramétrer à notre guise.
On peut également trouver beaucoup d'exemples et de ressources sur [Docker Hub](https://hub.docker.com/) 

Pour l'exemple on va ici récupérer ce template
```yaml
services:
	web:
	  image: nginx
	  volumes:
	   - ./templates:/etc/nginx/templates
	  restart: unless-stopped
	  ports:
	   - "8080:80"
	  environment:
	   - NGINX_HOST=foobar.com
	   - NGINX_PORT=80
```

Petite explication des différents champs :

  - **image**: la référence de l'image enregistré sur le docker Hub, vous n'aurez jamais à changer ce champ
  
  - **volumes**: Vous aurez parfois besoin de modifier des fichiers de configuration ou autres une fois votre docker lancé, pour se faire, il faut `binder` un répertoire sur votre machine avec un répertoire présent dans le système du docker
  
	  - ici le dossier `templates` sera créé à la racine ou est présente votre fichier `compose.yml` et sera relié au dossier `/etc/nginx/templates` 
	  
	  - Chaque fois que vous modifierez ou ajouterez des fichiers dans votre dossier local `templates` celui ci sera pris en compte dans le docker (avec besoin de redémarrer le docker ou non selon les cas)
	
  - **ports**: Ici on bind les ports interne au docker avec le port sur lequel sera présent votre service sur votre serveur
  
	  - `8080:80` signifie donc que votre service sera disponible sur le port `8080` de votre serveur
	  
	  - Si conflits, vous pouvez changer le port de gauche, mais ne jamais toucher au port de droite, propre au docker
	
  - **restart**: Indispensable de le mettre a `unless-stopped` ou `always` si vous voulez que votre docker au démarrage de votre machine
  
  - **environment**: des variables spécifiques au conteneur docker, on peut facilement trouver sur les différentes documentations les valeurs possibles à configurer 

## Création du fichier

Pour créer le fichier on peut suivre cette suite de commande :

```bash
mkdir nginx
cd nginx
sudo nano compose.yml
```

---

# Lancement et test du service

On peut désormais lancer notre docker avec la commande :

```bash
docker compose up -d
```

Si tout va bien, vous pouvez donc vous connecter sur votre navigateur à l'adresse 

- `localhost:<port>` (exemple : `localhost:8080`)
ou
- `<adresse tailscale du serveur>:<port>` (exemple : `100.116.11.79:8080`)

# Diagnoctic et debugging

Ici au lancement de mon service, j'ai un message d'erreur très explicite :

![Pasted image 20260307103640](/__images/Pasted%20image%2020260307103640.png)

Il indique sur le port 8080 est déjà alloué sur mon serveur
Je vais donc modifier le champ ports de mon `compose.yml` pour le passer à un port disponible : `8088:80` 


**Youpi !** Mon service est bien disponible désormais

![Pasted image 20260307104107](/__images/Pasted%20image%2020260307104107.png)

Parfois le service ne fonctionne pas du premier coup mais on ne sait pas vraiment pourquoi.

Pour affiner notre enquête on peut aller consulter les logs du conteneur docker avec la commande suivante : 

```bash
docker logs nginx-web-1
```

Je n'ai pas inventé le nom du conteneur, je l'ai trouvé facilement grâce à la tabulation.

Si on ne le trouve pas avec la tabulation on le trouve en faisant la commande : 

```bash
docker ps
```

![Pasted image 20260307104401](/__images/Pasted%20image%2020260307104401.png)

Le résultat du `docker logs` s'affiche, on peut continuer notre enquête ! 
![Pasted image 20260307104509](/__images/Pasted%20image%2020260307104509.png)

---

# Intégration du service dans un tunnel CloudFlare

On veut désormais rendre disponible notre service sous notre nom de domaine et en `https`, on va donc l'intégrer à notre tunnel Cloudflare.

*Prérequis: avoir déjà un tunnel cloudflare installé et configuré sur notre serveur*
[Créer son propre tunnel Cloudflare](Créer%20son%20propre%20tunnel%20Cloudflare.md)
## Déclaration de la route dans le dashboard Cloudflare

![Pasted image 20260307105445](/__images/Pasted%20image%2020260307105445.png)

Sur le site Web de cloudflare, il faut aller dans **DNS -> Records (enregistrements)** pour venir y ajouter un nouvel enregistrement.

- Type: **CNAME**
- Name: Le nom que l'on souhaite (il sera préfixé sur notre nom de domaine, exemple : nginx.monserveur.com )
- Target: copier-coller la même information que sur les autres

## Déclaration dans le fichier de configuration Cloudflare

On va maintenant définir sur notre serveur, quel service doit être redirigé vers notre nouvelle URL `nginx.monserveur.com`

Pour se faire on va venir taper cette commande 

```bash
sudo nano ~/.cloudflared/config.yml
```

On vient ensuite modifier le fichier comme ci-dessous
```yaml
tunnel: <TUNNEL-ID>
credentials-file: <PATH-TO-CREDENTIALS-FILES>

ingress:
  - hostname: nginx.remzik.com
    service: http://localhost:8088
  - service: http_status:404

```

En ajoutant en *hostname* la route désirée, et *service* l'adresse locale du service

On peut finalement venir redémarrer notre tunnel cloudlare afin que les modifications soient prises en compte : 

```bash
sudo systemctl restart cloudflared
```

**Victoire !** Notre site est bien disponible sur notre url, et en https ! 
![Pasted image 20260307111444](/__images/Pasted%20image%2020260307111444.png)