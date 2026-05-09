
[Vaultwarden](https://github.com/dani-garcia/vaultwarden) est un fork open-source du gestionnaire de mot de passe [Bitwarden](https://bitwarden.com/fr-fr/)

### Installation

On peut facilement l'auto-héberger dans un docker avec la configuration suivante :

```yaml
services:
  vaultwarden:
      container_name: vaultwarden
      volumes:
          - /srv/appdata/vaultwarden:/data
     #environment:
         #- SIGNUPS_ALLOWED=false
      image: 'vaultwarden/server:latest'
      ports:
        - "8081:80"
      restart: always
```

Une fois configuré et déployé via le flux : [Créer son propre tunnel Cloudflare](../configuration/Créer%20son%20propre%20tunnel%20Cloudflare.md) [Procédure d'installation d'un nouveau service](../configuration/Procédure%20d'installation%20d'un%20nouveau%20service.md)

### Extension navigateur

Il ne reste plus qu'a installer [l'application de bureau ou l'extension de navigateur bitwarden](https://bitwarden.com/fr-fr/download/)
Dans l'extension, choisir `self-hosted`
![Pasted image 20260310174221](/__images/Pasted%20image%2020260310174221.png)

Et indiquer l'url de votre VaultWarden.

Sur l'extension on peut également générer des mots de passes, et choisir les conditions de génération pour ceux-ci

![Pasted image 20260310180003](/__images/Pasted%20image%2020260310180003.png)

### Interface Web et outils

Sur l'interface Web de Vaultwarden, on peut retrouver beaucoup d'outils et de rapports nous aidant à gérer nos mots de passe réutilisés, exposés, faibles etc.

![Pasted image 20260310174758](/__images/Pasted%20image%2020260310174758.png)


### Intégrité des données et sauvegarde

Il est fortement recommandé d'effectuer des sauvegardes vers un cloud ou solution de stockage extérieure, afin de maintenir l'intégrité des données si votre serveur venait à dysfonctionner.

On peut retrouver [ici](https://github.com/dani-garcia/vaultwarden/wiki/Backing-up-your-vault) la recommandation de VaultWarden.

Une commande est prévu pour faire un backup de la base de données (SQLITE) en docker :

```bash
docker exec -it vaultwarden /vaultwarden backup
```

On peut donc facilement venir faire un script de sauvegarde comme ci-dessous :

```bash
#!/bin/bash

set -e

DATE=$(date +"%Y-%m-%d_%H-%M-%S")
BACKUP_DIR="/tmp/vaultwarden-backup-$DATE"
ARCHIVE="/tmp/vaultwarden-backup-$DATE.zip"

mkdir -p "$BACKUP_DIR"

# Backup SQLite
docker exec vaultwarden /vaultwarden backup

# Récupérer le dernier backup généré
LATEST_DB=$(ls -t /srv/appdata/vaultwarden/db_*.sqlite3 | head -n1)

cp "$LATEST_DB" "$BACKUP_DIR/"

# Clés RSA
cp /srv/appdata/vaultwarden/rsa_key* "$BACKUP_DIR/"

# Compression
cd /tmp
zip -r "$ARCHIVE" "$(basename "$BACKUP_DIR")"

# Upload rclone
sudo rclone copy "$ARCHIVE" drime:backup/vaultwarden/

# Nettoyage
rm -rf "$BACKUP_DIR"
rm "$ARCHIVE"
rm -f "$LATEST_DB"

```

Ce script requiert d'avoir une configuration [rclone](../configuration/sauvegardes/rclone.md) déjà en place.

On pourra ensuite venir configurer ce script dans [cron](../configuration/sauvegardes/cron.md) pour le faire tourner à occurence régulière.