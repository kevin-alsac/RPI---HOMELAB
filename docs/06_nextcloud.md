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
