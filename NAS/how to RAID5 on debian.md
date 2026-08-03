
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

#Créer dossier puis le monter
mkdir data
sudo mount /dev/md0 /data

#Enregistrer la conf
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf

#Faire en sorte que ça se monte au démarrage
echo "/dev/mdO /data ext4 rw,nofail,relatime,x-systemd.device-timeout=20s,defaults 0 2" >> /etc/fstab

echo "UUID=bbc1fa53-d6a4-4428-b182-c1a9ef80bcc5 /data ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab

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