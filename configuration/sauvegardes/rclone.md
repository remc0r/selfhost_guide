[rclone](https://rclone.org/) est un outil répandu pour automatiser l'envoi de données à un serveur distant, il est en effet compatible avec grand nombre de cloud.

## installation

Pour installer sa dernière version on viendra taper cette commande : 

 ```bash
 sudo -v ; curl https://rclone.org/install.sh | sudo bash
 ```

### configuration 

on viendra ensuite taper cette commande afin de paramétrer notre serveur distant : 

```bash
sudo rclone config
```
*J'utilise ici rclone en mode sudo, afin que l'accès aux différents fichiers aussi en mode sudo, ce n'est peut-être pas une pratique recommandable/optimisée*

le reste de la configuration dépendra de votre fournisseur de serveur distant.

Voici les exemples des docs respectives pour [Drime](https://drime.cloud/fr/blog-posts/drime-now-supports-native-rclone-integration) et [pCloud](https://rclone.org/pcloud/)

## lancement

pour lancer ensuite une sauvegarde nous avons deux choix :

- **rclone copy** : scanne le dossier local et copie tout les nouveaux fichiers sur le dossier distant sans en supprimer
- **rclone sync** : scanne le dossier local et synchronise avec le dossier distant, donc si un fichier a été supprimé sur le dossier local, il sera supprimé du dossier distant afin que les deux dossier soient synchronisés

pour ma part, pour la sauvegarde externes des données sensibles de mon serveur, je préfère user du **rclone copy**

voici un exemple de commande que je lance 

```bash
sudo /usr/bin/rclone copy /path/to/data drime:distant/path --drime-upload-cutoff 10M --log-file=/var/log/rclone/rclone-$(date +\%F).log --log-level INFO --stats 1m --stats-one-line
```

### explication de la commande et ses paramètres

#### `/usr/bin/rclone`

Chemin absolu vers l’exécutable `rclone`.

Utiliser le chemin complet permet :

- d’éviter les problèmes de variable `PATH` ;
- d’assurer que le bon binaire est utilisé ;
- d’être plus fiable dans les scripts automatisés (cron, systemd).

---
#### `copy`

Commande `rclone` utilisée pour copier des fichiers.

##### Fonctionnement

- copie les fichiers du dossier source vers la destination ;
- ne supprime rien côté destination ;
- ignore les fichiers déjà synchronisés si inchangés.

##### Différence avec `sync`

- `copy` → ajoute/met à jour uniquement ;
- `sync` → rend la destination identique à la source (suppression possible).

---

#### `/path/to/data`

Chemin local source à copier.

Exemple :

```
/home/user/documents
```

C’est le dossier ou fichier présent sur la machine locale.

---

#### `drime:distant/path`

Destination distante configurée dans `rclone`.

##### Décomposition

- `drime`  
    → nom du remote défini dans la configuration `rclone`.
- `distant/path`  
    → chemin distant dans le stockage cloud.

##### Exemple

Si le remote `drime` pointe vers un cloud :

```
drime:backups/serveur1
```

alors les fichiers seront copiés dans le dossier `backups/serveur1`.

---

### Options

#### `--drime-upload-cutoff 10M`

Définit le seuil de taille à partir duquel `rclone` change de méthode d’upload.

##### Ici

```
10M
```

signifie :

- les fichiers ≤ 10 Mo utilisent un upload simple ;
- les fichiers > 10 Mo utilisent un upload multipart/chunked.

##### Intérêt

Permet :

- d’optimiser les performances ;
- d’améliorer la reprise sur erreur ;
- de réduire la consommation mémoire.

##### Remarque

Cette option est spécifique au backend `drime`.

---

#### `--log-file=/var/log/rclone/rclone-$(date +\%F).log`

Enregistre les logs dans un fichier.

##### Exemple de résultat

```
/var/log/rclone/rclone-2026-05-09.log
```

##### Partie dynamique

```
$(date +\%F)
```

génère automatiquement la date du jour au format :

```
YYYY-MM-DD
```

##### Utilité

Permet :

- d’avoir un fichier de log par jour ;
- de faciliter le débogage ;
- d’archiver les exécutions.

##### Pourquoi `\%F` ?

Le `%` est échappé (`\%`) pour éviter des problèmes dans certains contextes comme :

- cron ;
- scripts shell.

---

#### `--log-level INFO`

Définit le niveau de détail des logs.

##### Niveau `INFO`

Affiche :

- les opérations importantes ;
- les fichiers transférés ;
- les statistiques ;
- les erreurs ;
- les avertissements.

##### Autres niveaux possibles

- `ERROR`
- `WARNING`
- `INFO`
- `DEBUG`

##### Usage conseillé

- `INFO` → production normale ;
- `DEBUG` → diagnostic détaillé.

---

#### `--stats 1m`

Affiche les statistiques toutes les 1 minute.

##### Exemple de statistiques

- nombre de fichiers transférés ;
- volume transféré ;
- vitesse ;
- temps restant ;
- erreurs éventuelles.

##### Ici

```
1m
```

= affichage toutes les 60 secondes.

---

#### `--stats-one-line`

Affiche les statistiques sur une seule ligne.

##### Sans cette option

Les stats prennent plusieurs lignes.

##### Avec cette option

Sortie compacte, pratique pour :

- les logs ;
- les journaux système ;
- la lecture via `tail -f`.





