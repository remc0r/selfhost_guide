

```bash
ssh user@ip_du_serveur
```

```bash
pwd # où suis-je ?  
ls -lah # lister fichiers  
cd /etc # changer de dossier
```

```bash
cp fichier.txt copie.txt  
mv fichier.txt dossier/  
rm fichier.txt  
mkdir dossier  
nano fichier.txt
```

```bash
ls -l  
chmod 755 script.sh  
chown user:user fichier
```

Explication du sudo et des systèmes de groupes

```bash
systemctl status nginx  
systemctl start nginx  
systemctl stop nginx  
systemctl restart nginx  
systemctl enable nginx
```

## Docker

compose.yaml
``` yaml
services:  
nginx:  
image: nginx  
ports:  
- "8080:80"  
volumes:  
- ./html:/usr/share/nginx/html
```

```bash
docker compose up -d
docker compose down
docker logs jellyfin
```

## Maintenance et mise à jour

```bash
sudo apt update  
sudo apt upgrade
```


# Ressources externes pour appréhender la ligne de commande 

- [https://blog.stephane-robert.info/docs/admin-serveurs/linux/commandes-introduction/](https://blog.stephane-robert.info/docs/admin-serveurs/linux/commandes-introduction/ "https://blog.stephane-robert.info/docs/admin-serveurs/linux/commandes-introduction/") 

- [https://assets.hostinger.com/content/tutorials/pdf/Linux-Commands-Cheatsheet-FR.pdf](https://assets.hostinger.com/content/tutorials/pdf/Linux-Commands-Cheatsheet-FR.pdf "https://assets.hostinger.com/content/tutorials/pdf/Linux-Commands-Cheatsheet-FR.pdf")

- [https://www.geeksforgeeks.org/linux-unix/linux-commands-cheat-sheet/](https://www.geeksforgeeks.org/linux-unix/linux-commands-cheat-sheet/ "https://www.geeksforgeeks.org/linux-unix/linux-commands-cheat-sheet/")
