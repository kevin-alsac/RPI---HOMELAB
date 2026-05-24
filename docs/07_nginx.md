# NGINX - Reverse proxy HTTPS pour Nextcloud

## Mise en place du reverse proxy NGINX

### Création du dossier

```bash
mkdir -p ~/docker/nextcloud/nginx
```

---

### Création du fichier de configuration

```bash
nano ~/docker/nextcloud/nginx/default.conf
```

Contenu :

```nginx
server {
    listen 443 ssl;
    server_name homelab-pi.xxxxxxxx.ts.net;

    ssl_certificate     /certs/homelab-pi.xxxxxxxx.ts.net.crt;
    ssl_certificate_key /certs/homelab-pi.xxxxxxxx.ts.net.key;

    client_max_body_size 10G;

    location / {
        proxy_pass http://nextcloud:80;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

---

### Ajout du service NGINX dans compose.yaml

```yaml
nginx:
  image: nginx:alpine
  container_name: nextcloud-nginx
  restart: unless-stopped
  depends_on:
    - nextcloud
  ports:
    - "443:443"
  volumes:
    - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    - /srv/secure-data/certs:/certs:ro
```

---

### Redémarrage de la stack

```bash
cd ~/docker/nextcloud
docker compose up -d
```

Vérification :

```bash
docker container ls
```

Conteneurs attendus :

```text
nextcloud
nextcloud-postgres
nextcloud-redis
nextcloud-nginx
```

---

### Vérification des logs NGINX

```bash
docker logs nextcloud-nginx
```

Le message suivant est normal :

```text
can not modify /etc/nginx/conf.d/default.conf (read-only file system?)
```

Le fichier est monté en lecture seule volontairement.

---

### Vérification de l'écoute HTTPS

```bash
sudo ss -tlnp | grep 443
```

Résultat attendu :

```text
0.0.0.0:443
[::]:443
```

---

## Configuration Nextcloud derrière NGINX

### Ajout du domaine approuvé

Vérification :

```bash
docker exec -u www-data nextcloud php occ config:system:get trusted_domains
```

Ajout du domaine Tailscale :

```bash
docker exec -u www-data nextcloud php occ config:system:set trusted_domains 1 --value=homelab-pi.xxxxxxxx.ts.net
```

---

### Forcer HTTPS dans Nextcloud

```bash
docker exec -u www-data nextcloud php occ config:system:set overwriteprotocol --value=https
```

---

### Déclarer le reverse proxy

```bash
docker exec -u www-data nextcloud php occ config:system:set trusted_proxies 0 --value=nginx
```

---

## Résultat final

Accès Nextcloud :

```text
https://homelab-pi.xxxxxxxx.ts.net
```

Architecture :

```text
Client
   │
HTTPS (443)
   │
NGINX
   │
HTTP interne Docker
   │
Nextcloud
   │
PostgreSQL
   │
Redis
```

Le trafic est maintenant :

- chiffré via HTTPS ;
- protégé par Tailscale ;
- servi par NGINX ;
- stocké sur la partition LUKS chiffrée.
