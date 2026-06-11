
```bash
services:
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: gluetun
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
       - VPN_SERVICE_PROVIDER=mullvad
       - VPN_TYPE=wireguard
       - WIREGUARD_PRIVATE_KEY=
       - WIREGUARD_ADDRESSES=
       - SERVER_COUNTRIES=
       - FIREWALL_VPN_INPUT_PORTS=50000
    ports:
      - 8080:8080  
    volumes:
      - ./gluetun:/gluetun
    restart: unless-stopped

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    depends_on:
      - gluetun
    network_mode: "service:gluetun"
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
      - /mnt/ssd/media/music/inbox:/downloads/music
      - /mnt/ssd/media/books:/downloads/books
      - /media:/media
      - ./scripts:/scripts
      - ./logs:/downloads/logs
    restart: unless-stopped
```
