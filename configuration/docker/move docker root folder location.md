https://www.ibm.com/docs/en/z-logdata-analytics/5.1.0?topic=software-relocating-docker-root-directory

### ✅ 1. Déplacer Docker ailleurs (RECOMMANDÉ)

👉 typiquement vers ton gros disque :

/mnt/ssd

💡 C’est **la meilleure pratique** quand :

- tu fais du Docker intensif
- tu as une petite partition système

---

### Comment faire proprement

1. Stop Docker

```
sudo systemctl stop docker
```

2. Copier les données

```
# les slashes finaux sont importants : sans eux, rsync crée /mnt/ssd/docker/docker
sudo rsync -aP /var/lib/docker/ /mnt/ssd/docker/
```

3. Configurer Docker

Dans `/etc/docker/daemon.json` :

```
{  
  "data-root": "/mnt/ssd/docker"  
}
```

4. Renommer l’ancien dossier (backup)

```
sudo mv /var/lib/docker /var/lib/docker.bak
```

5. Redémarrer

```
sudo systemctl start docker
```

6. Vérifier :

```
docker info | grep "Docker Root Dir"
```

👉 Puis supprimer le backup quand OK :

```
sudo rm -rf /var/lib/docker.bak
```