# commandes utiles
```bash
# autoriser acces ssh local
sudo tailscale set --exit-node-allow-lan-access=true

# se co sur l'exit node
sudo tailscale up --exit-node=100.74.223.51

# se co sur l'exit node sans prendre le dns mullvad 
sudo tailscale up --exit-node=100.74.223.51 --accept-dns=false

# alias pour passer du mode dns mullvad a dns adguard
alias ts-normal="sudo tailscale up --reset"
alias ts-exit="sudo tailscale up --exit-node=100.74.223.51 --exit-node-allow-lan-access"

```

# docker

https://fathi.me/unlock-secure-freedom-route-all-traffic-through-tailscale-gluetun/

*compose.yml*
```yml
volumes:
  ts-data:

services:
  # For additional VPN service providers, see: https://github.com/qdm12/gluetun-wiki
  gluetun:
    image: qmcgaw/gluetun
    restart: unless-stopped
    container_name: gluetun-mullvad
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - VPN_SERVICE_PROVIDER=mullvad
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=
      - WIREGUARD_ADDRESSES=
      - DNS_ADDRESS=
      - SERVER_COUNTRIES=Switzerland
  tailscale-vpn-exit-node:
    image: tailscale/tailscale:latest
    container_name: tailscale-vpn-exit-node
    network_mode: service:gluetun
    environment:
      - TS_AUTHKEY=
      - TS_EXTRA_ARGS=--advertise-exit-node  # or --advertise-tags=tag:vpn
      - TS_STATE_DIR=/var/lib/tailscale
      - TS_HOSTNAME=vpn-exit-node
    volumes:
      - ts-data:/var/lib/tailscale
    devices:
      - /dev/net/tun:/dev/net/tun
    cap_add:
      - NET_ADMIN
      - NET_RAW
    restart: unless-stopped
    depends_on:
      gluetun:
        condition: service_healthy
```

