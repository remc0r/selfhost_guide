Afin de télécharger sans prendre le risque de dévoiler son IP publique, on peut facilement mettre en place un `docker compose` contenant deux services :

- qbittorent : un client de téléchargement de torrent
- gluetun : un outil qui permets d'encapsuler la connexion du container à travers un VPN

Pour mettre en place ce `docker compose` composé de deux services, en voici un exemple : 

```bash
services:
  gluetun:
    image: qmcgaw/gluetun
    container_name: gluetun
    cap_add:
      - NET_ADMIN   # Droits réseau pour VPN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - VPN_SERVICE_PROVIDER=protonvpn
      - VPN_TYPE=openvpn
      - OPENVPN_USER=<YOUR_OPENVPN_USER>
      - OPENVPN_PASSWORD=<YOUR_OPENVPN_PASSWORD>
      - SERVER_COUNTRIES=Netherlands
      - TZ=Europe/Paris
      - DNS_UPSTREAM_PLAIN_ADDRESSES=1.1.1.1:53,8.8.8.8:53
    ports:
      - 8080:8080     # Expose le Web UI de Gluetun (optionnel)
      - 6881:6881
      - 6881:6881/udp
    volumes:
      - ./gluetun:/gluetun
    restart: unless-stopped

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    depends_on:
      - gluetun
    network_mode: "service:gluetun"  # qBittorrent utilise le réseau VPN de Gluetun
    deploy:
      resources:
        limits:
          memory: 8000M
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Paris
      - WEBUI_PORT=8080
    volumes:
      - ./config:/config
      - /mnt/ssd/media/movies:/downloads/films
      - /mnt/ssd/media/series:/downloads/series
      - /mnt/ssd/media/music:/downloads/music
      - /mnt/ssd/media/books:/downloads/books
      - /media:/media
      - ./scripts:/scripts
      - ./logs:/downloads/logs
    restart: unless-stopped

```

Pour trouver vos identifiants OpenVPN sur proton il suffit de vous rendre à cet endroit : 
![697](../__images/Screenshot%20From%202026-04-23%2022-07-13.png)
## what's next ?

éxécuter un [scripts qbittorrent](scripts%20qbittorrent.md) personnalisé après le téléchargement de chaque torrent ! 