# Mon Homelab Raspberry Pi 5

Vue globale du projet, architecture cible et état d’avancement.

---

# Objectif

Construire un homelab fiable et sécurisé sur Raspberry Pi 5 avec :

- accès distant sécurisé ;
- services conteneurisés Docker ;
- stockage chiffré LUKS ;
- HTTPS ;
- Nextcloud ;
- sauvegardes automatiques vers NAS.

---

# Matériel utilisé

| Matériel           | Détails                     |
| ------------------ | --------------------------- |
| Raspberry Pi       | Raspberry Pi 5              |
| RAM                | 16 Go                       |
| Stockage principal | SSD NVMe 1 To               |
| OS                 | Raspberry Pi OS Lite 64-bit |
| Réseau             | Ethernet recommandé         |
| Accès distant      | Tailscale                   |
| Sauvegardes        | NAS TerraMaster             |

---

# Système d’exploitation

Image utilisée :

```text
Raspberry Pi OS Lite (64-bit)
A port of Debian Trixie with no desktop environment
Compatible with Raspberry Pi 3/4/400/5
```

Pourquoi cette version :

- légère ;
- sans interface graphique;
- idéale pour serveur et Docker ;
- consommation de ressources réduite ;
- surface d’attaque plus faible ;
- consommation RAM réduite ;
- maintenance simplifiée.

---

# Architecture du stockage

| Partition   | Rôle                    |
| ----------- | ----------------------- |
| `nvme0n1p1` | Boot Raspberry Pi       |
| `nvme0n1p2` | Raspberry Pi OS         |
| `nvme0n1p3` | Partition chiffrée LUKS |

Le système reste séparé des données sensibles.

Les données Docker et Nextcloud seront stockées dans la partition chiffrée.

---

# Réseau

## Recommandation importante

Configurer :

- une IP fixe ;
  ou
- une réservation DHCP sur la box/routeur.

Pourquoi :

- éviter que l’adresse IP du Raspberry Pi change ;
- garder un accès SSH stable ;
- éviter les problèmes avec Docker, Nextcloud et Tailscale.

---

# Exemple d’architecture réseau

```text
Internet
   │
   ▼
Box / Routeur
   │
   ├── NAS
   │
   └── Raspberry Pi 5
         ├── Docker
         ├── Nextcloud
         ├── Tailscale
         └── Stockage LUKS
```

---

# Structure du projet

```text
mon-homelab-rpi5/
├── 📄 README.md
│
└── 📂 docs/
    ├── 📄 01_chiffrement_luks.md
    ├── 📄 02_connexion_ssh.md
    ├── 📄 03_deverouillage_manuel_luks.md
    ├── 📄 04_reseau_secu.md
    ├── 📄 05_docker_apps.md
    ├── 📄 06_nextcloud.md
    └── 📄 07_sauvegardes.md
```

---

# Guides

- [ ] [01 - Chiffrement LUKS](docs/01_chiffrement_luks.md)
- [ ] [02 - Connexion SSH](docs/02_connexion_ssh.md)
- [ ] [03 - Déverrouillage manuel LUKS](docs/03_deverouillage_manuel_luks.md)
- [ ] [04 - Réseau et sécurité](docs/04_reseau_secu.md)
- [ ] [05 - Docker et applications](docs/05_docker_apps.md)
- [ ] [06 - Nextcloud](docs/06_nextcloud.md)
- [ ] [07 - Sauvegardes](docs/07_sauvegardes.md)

---
