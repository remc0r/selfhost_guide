[Traefik](https://traefik.io/) est un reverse proxy : il reçoit tout le trafic HTTP(S) et le redirige vers les bons conteneurs selon le nom de domaine. [CrowdSec](https://www.crowdsec.net/) analyse les logs et bannit les IP malveillantes, et [Cloudflare](https://www.cloudflare.com/) gère le DNS et les certificats HTTPS.

Architecture finale :

```
Internet ──► Traefik (:80/:443) ──► middleware crowdsec ──► conteneur (vaultwarden, jellyfin...)
                 │                        ▲
                 │ access.log (json)      │ décisions (ban)
                 ▼                        │
             CrowdSec (analyse des logs) ─┘
```

- **Cloudflare** : DNS + certificats Let's Encrypt via le *DNS challenge* (pas besoin d'exposer le port 80 pour valider le certificat, et les wildcards sont possibles)
- **Traefik** : routage par labels Docker + HTTPS automatique
- **CrowdSec** : détecte (lecture des logs) et Traefik bloque (plugin bouncer)

> [!note]
> Prérequis : Docker installé ([introduction & installation](../configuration/docker/introduction%20&%20installation.md)), un nom de domaine dont les DNS sont gérés par Cloudflare, et les ports 80/443 redirigés vers le serveur.

---
## 1. Réseau Docker partagé

Tous les conteneurs exposés via Traefik (et CrowdSec) doivent être sur le même réseau :

```bash
docker network create proxy
```

---
## 2. Cloudflare

### Token API

Pour que Traefik puisse créer l'enregistrement DNS de validation (`_acme-challenge`), il lui faut un token :

1. Cloudflare → *My Profile* → *API Tokens* → *Create Token*
2. Template **Edit zone DNS**
3. Permissions : `Zone / DNS / Edit` (et `Zone / Zone / Read`)
4. Zone resources : limiter à votre domaine
5. Copier le token (il n'est affiché qu'une fois)

### Enregistrements DNS

Créer un enregistrement `A` par service (ou un wildcard `*`) pointant vers l'IP publique de la maison.

Les enregistrements sont en **DNS only (nuage gris)** : Cloudflare ne sert que de DNS, le trafic arrive directement sur votre IP et Traefik voit l'IP réelle des visiteurs, ce qui est nécessaire pour que CrowdSec bannisse la bonne IP.

> [!warning]
> Ne pas passer les enregistrements en *Proxied* (nuage orange) sans configurer `forwardedHeaders.trustedIPs` dans Traefik avec les [IP Cloudflare](https://www.cloudflare.com/ips/) : sinon CrowdSec ne verrait que des IP Cloudflare.

Si votre IP change régulièrement, voir un client DDNS (ex : `favonia/cloudflare-ddns`).

---
## 3. Traefik

```
traefik/
├── compose.yml
├── traefik.yml        # config statique
├── dynamic.yml        # config dynamique (middlewares)
├── acme.json          # certificats (chmod 600)
├── logs/              # access.log lu par CrowdSec
└── .env
```

```bash
cd traefik
touch acme.json && chmod 600 acme.json
mkdir logs
```

### `.env`

```bash
CF_DNS_API_TOKEN=<token cloudflare>
```

### `compose.yml`

```yaml
services:
  traefik:
    image: traefik:v3
    container_name: traefik
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      # dashboard accessible uniquement depuis une IP privée (ici Tailscale)
      - "<IP-PRIVEE>:8888:8080"
    environment:
      - CF_DNS_API_TOKEN=${CF_DNS_API_TOKEN}
    labels:
      - traefik.enable=true
      # Dashboard local
      - traefik.http.routers.traefik-local.rule=Host(`<IP-PRIVEE>`)
      - traefik.http.routers.traefik-local.entrypoints=local
      - traefik.http.routers.traefik-local.service=api@internal
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yml:/etc/traefik/traefik.yml:ro
      - ./dynamic.yml:/etc/traefik/dynamic.yml:ro
      - ./acme.json:/acme.json
      - ./logs:/var/log/traefik
    networks:
      - proxy

networks:
  proxy:
    external: true
```

> [!warning]
> Ne jamais exposer le dashboard (`8080`) sur `0.0.0.0`. Ici il n'écoute que sur une IP privée. De même, le socket Docker est monté en lecture seule.

### `traefik.yml` (config statique)

```yaml
api:
  dashboard: true

# Logs d'accès en JSON : c'est ce que CrowdSec va analyser
accessLog:
  filePath: "/var/log/traefik/access.log"
  format: json

entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
  websecure:
    address: ":443"
  local:
    address: ":8080"

providers:
  docker:
    exposedByDefault: false   # seuls les conteneurs avec traefik.enable=true sont exposés
  file:
    filename: /etc/traefik/dynamic.yml

certificatesResolvers:
  cloudflare:
    acme:
      email: <VOTRE-EMAIL>
      storage: /acme.json
      dnsChallenge:
        provider: cloudflare

# Plugin bouncer CrowdSec
experimental:
  plugins:
    crowdsec:
      moduleName: github.com/maxlerebourg/crowdsec-bouncer-traefik-plugin
      version: v1.3.5
```

> [!tip]
> Pour tester sans risquer le rate-limit de Let's Encrypt, ajouter temporairement sous `acme:`
> `caServer: https://acme-staging-v02.api.letsencrypt.org/directory` (et vider `acme.json` en repassant en production).

---
## 4. CrowdSec

### `compose.yml`

```yaml
services:
  crowdsec:
    image: crowdsecurity/crowdsec:latest
    container_name: crowdsec
    restart: unless-stopped
    expose:
      - "8080"                      # API locale (LAPI), utilisée par le bouncer
    ports:
      - "127.0.0.1:8181:8080"       # optionnel : pour utiliser cscli depuis l'hôte
    volumes:
      - ./data:/var/lib/crowdsec/data
      - ./etc:/etc/crowdsec
      # Logs Traefik (en lecture seule)
      - ../traefik/logs:/var/log/traefik:ro
      # Logs SSH de l'hôte (optionnel)
      - /var/log/auth.log:/var/log/auth.log:ro
      - /var/log/syslog:/var/log/syslog:ro
    environment:
      - GID=1000
      - COLLECTIONS=crowdsecurity/traefik crowdsecurity/http-cve crowdsecurity/base-http-scenarios crowdsecurity/sshd crowdsecurity/linux
    networks:
      - proxy

networks:
  proxy:
    external: true
```

Les **collections** sont des paquets de parsers (qui savent lire un format de log) et de scénarios (qui détectent un comportement : brute-force, scan de vulnérabilités, etc.).

### Indiquer à CrowdSec quels logs lire

> [!important]
> C'est l'étape la plus facile à oublier. Sans fichier d'acquisition, CrowdSec démarre sans erreur mais **ne lit aucun log** : aucune détection locale, seule la blocklist communautaire est appliquée.

Créer `crowdsec/etc/acquis.d/traefik.yaml` :

```yaml
filenames:
  - /var/log/traefik/access.log
labels:
  type: traefik
```

Et, si les logs SSH sont montés (sur Debian récent, `/var/log/auth.log` n'existe que si `rsyslog` est installé, sinon Docker crée un dossier vide à la place), `crowdsec/etc/acquis.d/ssh.yaml` :

```yaml
filenames:
  - /var/log/auth.log
labels:
  type: syslog
```

> [!tip]
> Le fichier `access.log` grossit sans limite : prévoir une rotation (`logrotate`).

> [!note]
> Si `etc/acquis.yaml` contient un fichier factice (`/does/not/exist`), le supprimer ou le laisser vide, il ne gêne pas mais ne sert à rien.

### Démarrage et vérification

```bash
docker compose up -d
docker exec crowdsec cscli metrics show acquisition
```

Le tableau doit afficher une ligne `file:/var/log/traefik/access.log` avec des *Lines read* qui augmentent quand on visite un service.

### Créer la clé du bouncer

```bash
docker exec crowdsec cscli bouncers add traefik-bouncer
```

Copier la clé affichée (elle ne sera plus visible ensuite).

---
## 5. Brancher le plugin dans Traefik

Dans `traefik/dynamic.yml` :

```yaml
http:
  middlewares:
    crowdsec:
      plugin:
        crowdsec:
          enabled: true
          crowdsecMode: live              # interroge la LAPI à chaque requête (mise en cache)
          crowdsecLapiScheme: http
          crowdsecLapiHost: crowdsec:8080 # nom du conteneur sur le réseau proxy
          crowdsecLapiKey: <CLE-DU-BOUNCER>
          defaultDecisionSeconds: 60
          remediationStatusCode: 403
```

> [!warning]
> `dynamic.yml` contient la clé du bouncer : ne pas le commiter. On versionne un `dynamic.yml.example` avec `REPLACE_WITH_REAL_LAPI_KEY` et on ajoute `dynamic.yml`, `.env` et `acme.json` au `.gitignore`.

Démarrer Traefik :

```bash
cd traefik && docker compose up -d
docker logs traefik
```

---
## 6. Protéger un service

Il suffit d'ajouter les labels sur le conteneur. Exemple avec Vaultwarden :

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    volumes:
      - /srv/appdata/vaultwarden:/data
    labels:
      - traefik.enable=true
      - traefik.http.routers.vault.rule=Host(`vault.mondomaine.com`)
      - traefik.http.routers.vault.entrypoints=websecure
      - traefik.http.routers.vault.tls.certresolver=cloudflare
      - traefik.http.routers.vault.middlewares=crowdsec@file
      - traefik.http.services.vault.loadbalancer.server.port=80
    networks:
      - proxy

networks:
  proxy:
    external: true
```

- `certresolver=cloudflare` : Traefik demande le certificat automatiquement
- `middlewares=crowdsec@file` : le `@file` indique que le middleware vient de `dynamic.yml`
- Pas de `ports:` : le service n'est joignable que via Traefik

Plusieurs middlewares se chaînent avec une virgule, par exemple `crowdsec@file,authentik@file` pour ajouter une authentification SSO ([Authentik](https://goauthentik.io/)) derrière le filtrage CrowdSec.

---
## 7. Durcissement

### Rate limit et headers de sécurité

Deux middlewares supplémentaires dans `dynamic.yml` :

```yaml
http:
  middlewares:
    rate-limit:
      rateLimit:
        average: 100   # requêtes par seconde en moyenne, par IP
        burst: 200

    security-headers:
      headers:
        frameDeny: true                              # interdit l'affichage dans un iframe (clickjacking)
        contentTypeNosniff: true                     # le navigateur ne devine plus le type des fichiers
        referrerPolicy: strict-origin-when-cross-origin
        stsSeconds: 15552000                         # HSTS : HTTPS obligatoire pendant 180 jours
```

Ils se chaînent aux labels des services :

```yaml
      - traefik.http.routers.vault.middlewares=crowdsec@file,rate-limit@file,security-headers@file
```

- **rate-limit** : limite avant détection. CrowdSec bannit après coup, le rate limit freine tout de suite. Si une app envoie beaucoup de requêtes (synchronisation Nextcloud, apps mobiles), des erreurs `429` apparaissent : augmenter `average`.
- **security-headers** : protège le navigateur du visiteur, pas le serveur. À ne pas mettre sur les apps intégrées dans un iframe, ni sur celles qui envoient déjà leurs propres headers (Nextcloud).

> [!warning]
> Le HSTS oblige les navigateurs à refuser le HTTP sur ce domaine pendant 180 jours. Il faut donc que le renouvellement des certificats reste fonctionnel.

Vérification :

```bash
curl -sI https://vault.mondomaine.com | grep -i -E "strict|x-frame|nosniff|referrer"
```

### Ne pas exposer de ports inutiles

Tout port publié avec `"8191:8191"` écoute sur **toutes** les interfaces. Pour un service qui n'a pas besoin d'être public, préciser l'IP :

```yaml
    ports:
      - "<IP-PRIVEE>:8191:8191"    # ou 127.0.0.1
```

Lister les ports ouverts sur l'hôte :

```bash
ss -tlnp | grep -v '127.0.0.'
```

Un conteneur en `network_mode: host` (Home Assistant par exemple) contourne cette règle : il écoute directement sur l'hôte.

### Socket Docker

`/var/run/docker.sock` donne un contrôle total de la machine à qui peut y écrire. Le monter en lecture seule (`:ro`) limite les dégâts mais ne les supprime pas. Pour aller plus loin, placer un [docker-socket-proxy](https://github.com/Tecnativa/docker-socket-proxy) devant Traefik pour ne laisser passer que les requêtes en lecture.

### Rotation des logs

`access.log` grossit sans limite (plusieurs Go en quelques mois). Créer `/etc/logrotate.d/traefik` :

```
/chemin/vers/traefik/logs/access.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    postrotate
        docker kill -s USR1 traefik >/dev/null 2>&1 || true
    endscript
}
```

Le signal `USR1` demande à Traefik de rouvrir son fichier de log ; sans lui il continuerait d'écrire dans l'ancien. CrowdSec suit automatiquement le nouveau fichier. Test immédiat :

```bash
sudo logrotate -f /etc/logrotate.d/traefik
ls -lh traefik/logs
```

### Logs SSH sur Debian récent

Sans `rsyslog`, `/var/log/auth.log` n'existe pas (tout est dans journald) et Docker crée un **dossier** vide à la place lors du montage. Installer rsyslog, supprimer ces dossiers puis recréer le conteneur CrowdSec :

```bash
sudo apt install rsyslog
sudo systemctl enable --now rsyslog
sudo rmdir /var/log/auth.log /var/log/syslog   # seulement si ce sont des dossiers vides
sudo systemctl restart rsyslog
```

> [!note]
> Le plugin Traefik ne bloque que le HTTP. Pour que les bans CrowdSec s'appliquent aussi à SSH, il faut un bouncer firewall (`crowdsec-firewall-bouncer`). Si SSH n'est accessible que via un VPN (Tailscale), ce n'est pas indispensable.

---
## 8. Vérifier que ça fonctionne

```bash
# le bouncer est bien connecté (colonne "Last API pull" récente)
docker exec crowdsec cscli bouncers list

# les collections sont installées
docker exec crowdsec cscli collections list

# les logs sont lus et parsés
docker exec crowdsec cscli metrics show acquisition

# alertes et IP bannies
docker exec crowdsec cscli alerts list
docker exec crowdsec cscli decisions list
```

### Tester un ban

Bannir temporairement **une IP qui n'est pas la vôtre** (ex : celle d'un téléphone en 4G) :

```bash
docker exec crowdsec cscli decisions add --ip <IP-DE-TEST> --duration 5m
```

Depuis cette IP, le service doit répondre `403`. Avec `crowdsecMode: live` le ban est effectif en quelques secondes (le cache du plugin est de `defaultDecisionSeconds`). Pour lever le ban :

```bash
docker exec crowdsec cscli decisions delete --ip <IP-DE-TEST>
```

> [!warning]
> Ne testez pas avec votre propre IP, vous risquez de vous enfermer dehors. Ajoutez votre réseau local en whitelist avec `clientTrustedIPs` côté plugin.

---
## Dépannage

| Symptôme | Piste |
|---|---|
| Pas de certificat, erreur dans `docker logs traefik` | Token Cloudflare (permissions / zone), `acme.json` en `chmod 600` |
| `acquisition` vide dans `cscli metrics` | Fichier `acquis.d/*.yaml` absent, ou volume des logs mal monté |
| Bouncer absent de `cscli bouncers list` | Mauvaise clé dans `dynamic.yml`, ou Traefik et CrowdSec pas sur le même réseau `proxy` |
| Plugin non chargé | Version du plugin dans `traefik.yml`, accès internet du conteneur au démarrage |
| Erreurs `429` | `rate-limit` trop strict : augmenter `average` / `burst` |
| Une IP légitime est bannie | `docker exec crowdsec cscli decisions delete --ip <IP>` |

## Pour aller plus loin

- [Console CrowdSec](https://app.crowdsec.net/) : tableau de bord gratuit, on enrôle l'instance avec `docker exec crowdsec cscli console enroll <clé>`
- [Documentation du plugin bouncer](https://github.com/maxlerebourg/crowdsec-bouncer-traefik-plugin)
- [firewall et ports ouverts](firewall%20et%20ports%20ouverts.md)
