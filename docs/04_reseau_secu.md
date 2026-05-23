# Réseau et sécurité - UFW et Tailscale

## Objectif

Sécuriser l’accès réseau du Raspberry Pi 5 avec :

- un firewall Linux (`UFW`) ;
- un accès distant privé via `Tailscale` ;
- aucune ouverture de port sur la box Internet.

Cette architecture permet :

- de réduire fortement la surface d’attaque ;
- d’éviter l’exposition publique du serveur ;
- d’accéder au homelab depuis n’importe où via un réseau privé WireGuard.

---

# Architecture réseau choisie

```text
Internet
   │
   ▼
Tailscale privé
   │
   ▼
Raspberry Pi 5
   ├── SSH
   ├── Docker
   ├── Nextcloud
   └── Stockage LUKS
```

Aucun port n’est ouvert sur la box Internet.

---

# Pourquoi Tailscale only ?

Le projet utilise une architecture :

```text
Tailscale only
```

Cela signifie :

- pas de reverse proxy public ;
- pas de NAT/port forwarding ;
- pas de ports ouverts Internet ;
- accès uniquement via les appareils connectés au réseau Tailscale.

Avantages :

| Avantage                       | Impact                   |
| ------------------------------ | ------------------------ |
| Aucun port ouvert              | sécurité renforcée       |
| Réseau privé WireGuard         | accès distant sécurisé   |
| Configuration simplifiée       | moins de maintenance     |
| Pas de certificat HTTPS public | moins de complexité      |
| Pas de reverse proxy           | architecture plus légère |

---

# 1. Installation du firewall UFW

## Installer UFW

```bash
sudo apt update
sudo apt install ufw -y
```

---

# 2. Autoriser SSH

⚠️ Toujours autoriser SSH avant d’activer le firewall.

```bash
sudo ufw allow ssh
```

Cette commande autorise :

```text
22/tcp
```

---

# 3. Politique de sécurité

## Bloquer toutes les connexions entrantes

```bash
sudo ufw default deny incoming
```

---

## Autoriser les connexions sortantes

```bash
sudo ufw default allow outgoing
```

---

# 4. Activer le firewall

```bash
sudo ufw enable
```

Confirmer avec :

```text
y
```

---

# 5. Vérifier le statut

```bash
sudo ufw status verbose
```

Exemple :

```text
Status: active

22/tcp ALLOW IN Anywhere
```

---

# 6. Installation de Tailscale

Installer Tailscale :

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

---

# 7. Connecter le Raspberry Pi au réseau Tailscale

```bash
sudo tailscale up
```

Un lien de connexion apparaît :

```text
https://login.tailscale.com/...
```

Ouvrir ce lien dans un navigateur puis :

- se connecter ;
- autoriser le Raspberry Pi.

---

# 8. Vérifier la connexion Tailscale

```bash
tailscale status
```

---

# 9. Voir l’adresse IP Tailscale

```bash
tailscale ip -4
```

Exemple :

```text
100.x.x.x
```

Cette IP permet d’accéder au Raspberry Pi via le réseau privé Tailscale.

---

# 10. Installer Tailscale sur les autres appareils

Tous les appareils doivent être connectés au même réseau Tailscale :

- PC Windows ;
- téléphone Android/iPhone ;
- ordinateur portable ;
- etc.

Téléchargement :

- Windows :
  https://tailscale.com/download/windows

- Linux :
  https://tailscale.com/download/linux

- Android :
  https://play.google.com/store/apps/details?id=com.tailscale.ipn

- iPhone :
  https://apps.apple.com/us/app/tailscale/id1475387142

---

# 11. Connexion SSH via Tailscale

Exemple :

```bash
ssh pi_admin@100.x.x.x
```

Aucune ouverture de port Internet n’est nécessaire.

---

# 12. Simplifier les connexions SSH avec ~/.ssh/config

Sur le PC Windows :

```powershell
notepad $env:USERPROFILE\.ssh\config
```

Ajouter :

```sshconfig
Host pi_lan
    HostName 192.168.1.50
    User pi_admin
    IdentityFile ~/.ssh/id_ed25519

Host pi_wan
    HostName 100.x.x.x
    User pi_admin
    IdentityFile ~/.ssh/id_ed25519
```

---

# Utilisation

## Connexion LAN

```bash
ssh pi_lan
```

---

## Connexion Tailscale

```bash
ssh pi_wan
```

---

# 13. Contrôle d’accès Tailscale (Grants / ACL)

Par défaut, tous les appareils d’un réseau Tailscale peuvent communiquer entre eux.

Pour renforcer la sécurité du homelab, des règles d’accès (`grants`) ont été mises en place afin de :

- autoriser uniquement l’accès au Raspberry Pi ;
- empêcher les communications directes entre les autres appareils ;
- limiter les ports accessibles.

---

# Objectif du contrôle d’accès

Architecture souhaitée :

```text
PC Windows --------> Raspberry Pi
Téléphone ---------> Raspberry Pi

PC Windows -X-> Téléphone
Téléphone -X-> PC Windows
```

Les appareils clients peuvent accéder au serveur, mais pas communiquer entre eux.

---

# Configuration des Grants

Dans :

```text
Tailscale Admin Console
→ Access controls
```

Modification du fichier de policy :

```json
"grants": [
	{
		"src": ["adressemail@domaine.com"],
		"dst": ["100.x.x.x"],
		"ip": ["tcp:22", "tcp:443"]
	}
],
```

---

# Explication des règles

| Élément   | Description                 |
| --------- | --------------------------- |
| `src`     | utilisateur autorisé        |
| `dst`     | Raspberry Pi (IP Tailscale) |
| `tcp:22`  | autorise SSH                |
| `tcp:443` | autorise HTTPS              |

---

# Ports volontairement bloqués

Les autres protocoles et ports ne sont pas autorisés :

- ping (`ICMP`) ;
- communications entre clients ;
- services non explicitement déclarés.

Exemple :

```text
PC → Raspberry Pi : OK
Téléphone → Raspberry Pi : OK
PC ↔ Téléphone : BLOQUÉ
Ping : BLOQUÉ
```

---

# Approche sécurité utilisée

Cette configuration applique un principe de :

```text
Least privilege
```

C’est-à-dire :

> n’autoriser que les accès strictement nécessaires.

Cela permet :

- de réduire la surface d’attaque ;
- d’éviter les communications inutiles ;
- d’appliquer une logique Zero Trust simple ;
- de segmenter les accès du réseau privé Tailscale.

---

# Résultat final

Le Raspberry Pi dispose maintenant :

- d’un firewall actif ;
- d’un accès SSH sécurisé ;
- d’un accès distant privé via Tailscale ;
- d’aucune exposition publique sur Internet ;
- d’ACLs réseau restrictives ;
- d’un contrôle précis des appareils autorisés ;
- d’une architecture réseau simplifiée et sécurisée.
