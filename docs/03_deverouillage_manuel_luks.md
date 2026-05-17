# Montage manuel du volume chiffré LUKS

## Objectif

Monter manuellement une partition chiffrée LUKS après le démarrage du Raspberry Pi.

Cette approche permet :

- un boot stable du système ;
- de garder les données chiffrées séparées du système ;
- d’ouvrir le stockage sécurisé uniquement quand nécessaire.

---

# Principe de fonctionnement

Le Raspberry Pi démarre normalement sur :

```text
/dev/nvme0n1p2
```

Le stockage chiffré est séparé dans :

```text
/dev/nvme0n1p3
```

Quand le volume est déverrouillé :

```bash
sudo cryptsetup luksOpen /dev/nvme0n1p3 ssd_data
```

Linux crée :

```text
/dev/mapper/ssd_data
```

Ce volume déchiffré peut ensuite être monté dans un dossier du système.

---

# 1. Créer le point de montage

Créer le dossier :

```bash
sudo mkdir -p /srv/secure-data
```

## Important

Le dossier est créé initialement sur :

```text
/dev/nvme0n1p2
```

(c’est-à-dire le système Raspberry Pi OS)

Mais après le montage :

```bash
sudo mount /dev/mapper/ssd_data /srv/secure-data
```

le contenu affiché dans :

```text
/srv/secure-data
```

provient en réalité du volume chiffré :

```text
/dev/mapper/ssd_data
```

---

# 2. Donner les permissions

Adapter `pi_admin` selon votre utilisateur Linux :

```bash
sudo chown pi_admin:pi_admin /srv/secure-data
sudo chmod 700 /srv/secure-data
```

---

# 3. Ouvrir le volume LUKS

```bash
sudo cryptsetup luksOpen /dev/nvme0n1p3 ssd_data
```

Cette commande :

- demande le mot de passe LUKS ;
- déverrouille la partition chiffrée ;
- crée :

```text
/dev/mapper/ssd_data
```

---

# 4. Monter le volume

```bash
sudo mount /dev/mapper/ssd_data /srv/secure-data
```

## Explication

Cette commande rend le contenu du volume chiffré accessible dans :

```text
/srv/secure-data
```

Tous les fichiers créés dans ce dossier seront physiquement stockés dans :

```text
/dev/nvme0n1p3
```

et donc protégés par LUKS.

---

# 5. Vérifier le montage

```bash
df -h
```

Exemple attendu :

```text
/dev/mapper/ssd_data   879G   ...   /srv/secure-data
```

---

# 6. Vérifier le volume LUKS

```bash
sudo cryptsetup status ssd_data
```

Exemple :

```text
/dev/mapper/ssd_data is active
type: LUKS2
```

---

# 7. Créer un script de déverrouillage

Créer le fichier :

```bash
nano ~/unlock-secure.sh
```

Contenu :

```bash
#!/bin/bash

sudo cryptsetup luksOpen /dev/nvme0n1p3 ssd_data
sudo mount /dev/mapper/ssd_data /srv/secure-data

echo "Volume sécurisé monté."
```

---

# 8. Rendre le script exécutable

```bash
chmod 700 ~/unlock-secure.sh
```

## Explication

```text
700 = rwx------
```

Le script devient :

- exécutable ;
- accessible uniquement par votre utilisateur.

---

# 9. Créer un alias Bash

Éditer :

```bash
nano ~/.bashrc
```

Ajouter tout en bas :

```bash
alias unlock="~/unlock-secure.sh"
```

---

# 10. Recharger Bash

```bash
source ~/.bashrc
```

---

# 11. Utilisation

Après chaque reboot :

```bash
unlock
```

Le script :

1. ouvre le volume LUKS ;
2. demande le mot de passe LUKS ;
3. monte automatiquement le stockage sécurisé.

---

# 12. Fermer le volume

## Démonter le filesystem

```bash
sudo umount /srv/secure-data
```

---

## Fermer le volume LUKS

```bash
sudo cryptsetup luksClose ssd_data
```

---

# Résultat final

| Élément                | Rôle                                        |
| ---------------------- | ------------------------------------------- |
| `/dev/nvme0n1p2`       | système Raspberry Pi OS                     |
| `/dev/nvme0n1p3`       | partition chiffrée                          |
| `/dev/mapper/ssd_data` | volume déchiffré                            |
| `/srv/secure-data`     | point de montage accessible                 |
| `unlock`               | alias Bash vers le script de déverrouillage |

Cette architecture permet :

- un boot stable ;
- un stockage sécurisé ;
- un déverrouillage manuel simple ;
- et une séparation propre entre système et données.
