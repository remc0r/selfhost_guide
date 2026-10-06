
# RAID5
https://std.rocks/gnulinux_nas.html
```bash
#Créer la partition sur le premier disque
sudo gdisk /dev/sda

#Copier la config partition sur les deux autres disques
sudo sgdisk /dev/sda -R /dev/sdb
sudo sgdisk /dev/sda -R /dev/sdc

#Randomise les GUID des disques/partition
sudo sgdisk -G /dev/sdb
sudo sgdisk -G /dev/sdc

#Installer l'outil pour créer le RAID
sudo apt install mdadm

#Créer le RAID5
sudo mdadm --create --verbose /dev/md0   --level=5   --raid-devices=3   --name=remnas   /dev/sda1 /dev/sdb1 /dev/sdc1

# Vérifier l'avancement
cat /proc/mdstat

#Formater le RAID (possible pendant la synchronisation initiale)
sudo mkfs.ext4 /dev/md0

#Créer le point de montage puis monter
sudo mkdir -p /data
sudo mount /dev/md0 /data

#Enregistrer la conf, puis régénérer l'initramfs pour que le RAID soit assemblé au boot
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u

#Récupérer l'UUID du système de fichiers
sudo blkid /dev/md0

#Faire en sorte que ça se monte au démarrage (remplacer <UUID> par la valeur obtenue ci-dessus)
#On utilise l'UUID plutôt que /dev/md0 : le RAID peut être renommé /dev/md127 au redémarrage
echo "UUID=<UUID> /data ext4 defaults,nofail,x-systemd.device-timeout=20s 0 2" | sudo tee -a /etc/fstab

```

## Bonus

```bash
#Vérifier l'état des ports USB
lsusb -t

#Lister les disques
lsblk
```

# NFS

https://www.linuxtricks.fr/wiki/debian-installer-un-serveur-nfs

## Sur le NAS
```bash
#Installer utilitaire NFS
sudo apt install nfs-kernel-server

#Configurer les règles de partage
sudo nano /etc/exports 

# /data 192.168.1.0/24(rw,sync,no_subtree_check)

#Enable et/ou redemarrer
sudo systemctl enable --now nfs-server.service
sudo systemctl restart nfs-server

#Gestion des droits -> ici je mets le meme user que sur mon serveur pour éviter les conflits
sudo chown -R remcor:remcor /data
sudo chmod -R 775 /data

```

## Sur le serveur client
```bash
# Installer l'utilitaire NFS
sudo apt install nfs-common
# Monter le dossier NFS avec l'ip du NAS
sudo mount -t nfs 192.168.1.120:/data /media

# Rendre le montage persistant au démarrage
echo "192.168.1.120:/data /media nfs defaults,_netdev,nofail 0 0" | sudo tee -a /etc/fstab

```

## reprogrammer le mdcheck

### commande pour afficher le cron de vérification

```bash
systemctl cat mdcheck_start.timer
```

```bash

# Augmenter le temps maximum
sudo systemctl edit mdcheck_start.service
[Service]
Environment="MDADM_CHECK_DURATION=12 hours"

# Reprogrammer la fréquence
sudo systemctl edit mdcheck_start.timer

[Timer]
#Bien mettre la ligne a vide pour vider la configuration d'origine, sinon problème de doublons
OnCalendar=
OnCalendar=Mon *-*-1..7 3:00:00
RandomizedDelaySec=0

sudo systemctl daemon-reload

sudo systemctl restart mdcheck_start.timer
```