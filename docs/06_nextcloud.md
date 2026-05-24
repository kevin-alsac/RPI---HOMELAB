# Nextcloud - Installation et configuration

## Premier accès

Après le démarrage des conteneurs :

```bash
docker compose up -d
```

Vérifier :

```bash
docker container ls
```

Accéder à Nextcloud :

```text
http://100.x.x.x:8080
```

Créer :

- le compte administrateur ;
- le mot de passe administrateur.

Cliquer sur :

```text
Installer
```

et attendre la fin de l'initialisation.

---

## Vérifications après installation

Ouvrir :

```text
Paramètres d'administration
→ Vue d'ensemble
```

afin d'identifier les éventuels avertissements de configuration.

---

### Définir une fenêtre de maintenance

Configuration recommandée pour exécuter les tâches lourdes vers 03h00 du matin :

```bash
docker exec -u www-data nextcloud php occ config:system:set maintenance_window_start --type=integer --value=3
```

Vérification :

```bash
docker exec -u www-data nextcloud php occ config:system:get maintenance_window_start
```

Résultat attendu :

```text
3
```

---

### Définir la région par défaut

Permet à Nextcloud d'interpréter correctement les numéros de téléphone français.

```bash
docker exec -u www-data nextcloud php occ config:system:set default_phone_region --value=FR
```

Vérification :

```bash
docker exec -u www-data nextcloud php occ config:system:get default_phone_region
```

Résultat attendu :

```text
FR
```

---

## Authentification à deux facteurs (2FA)

Application activée :

```text
Two-Factor TOTP Provider
```

Chemin :

```text
Applications
→ Sécurité
→ Two-Factor TOTP Provider
```

Activation utilisateur :

```text
Avatar
→ Paramètres personnels
→ Sécurité
→ TOTP
```

Une application d'authentification (Aegis, Bitwarden Authenticator, etc.) peut être utilisée pour générer les codes à 6 chiffres.

---

## HTTPS avec Tailscale

### Activation des certificats HTTPS

Activation effectuée dans l'administration Tailscale.

Génération du certificat :

```bash
sudo tailscale cert homelab-pi.xxxxxxxxx.ts.net
```

Fichiers générés :

```text
homelab-pi.xxxxxxxx.ts.net.crt
homelab-pi.xxxxxxxx.ts.net.key
```

---

### Stockage sécurisé des certificats

Création du dossier :

```bash
sudo mkdir -p /srv/secure-data/certs
```

Déplacement des certificats :

```bash
sudo mv ~/homelab-pi*.crt /srv/secure-data/certs/
sudo mv ~/homelab-pi*.key /srv/secure-data/certs/
```

Vérification :

```bash
sudo ls -l /srv/secure-data/certs
```

Résultat :

```text
/srv/secure-data/certs
├── homelab-pi.xxxxxxxx.ts.net.crt
└── homelab-pi.xxxxxxxx.ts.net.key
```

---

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
