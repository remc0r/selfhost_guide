
https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/do-more-with-tunnels/local-management/create-local-tunnel/

## Installation du packet cloudflared

```bash
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-public-v2.gpg | sudo tee /usr/share/keyrings/cloudflare-public-v2.gpg >/dev/null
```

```bash
echo "deb [signed-by=/usr/share/keyrings/cloudflare-public-v2.gpg] https://pkg.cloudflare.com/cloudflared any main" | sudo tee /etc/apt/sources.list.d/cloudflared.list
```

```bash
sudo apt-get update && sudo apt-get install cloudflared
```

## Connexion à Cloudflare

```bash
cloudflared tunnel login
```

## Création du tunnel

```bash
cloudflared tunnel create <NAME>
```

## Création du fichier de configuration

```bash
sudo nano ~/.cloudflared/config.yml
```

```yaml
tunnel: <TUNNEL-ID>
credentials-file: <PATH-TO-CREDENTIALS-FILES>

ingress:
  - hostname: vault.remzik.com
    service: http://localhost:8081
  [...]
  - service: http_status:404

```

## Run as a service 

```bash
sudo cloudflared --config /home/<USER>/.cloudflared/config.yml service install
```
