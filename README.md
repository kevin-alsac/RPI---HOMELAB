# Chiffrement d’une partition SSD avec LUKS sur Raspberry Pi 5

## Objectif

Créer une partition chiffrée avec LUKS sur un SSD NVMe utilisé avec un Raspberry Pi 5, tout en conservant une partition système Raspberry Pi OS non chiffrée.

---

# Structure finale du disque

| Partition   | Rôle                          |
| ----------- | ----------------------------- |
| `nvme0n1p1` | Boot Raspberry Pi             |
| `nvme0n1p2` | Système Raspberry Pi OS       |
| `nvme0n1p3` | Partition de données chiffrée |

Une fois ouverte :

```text
/dev/mapper/ssd_data
```

correspondra au volume déchiffré utilisable normalement.

---

# Important avant de commencer

⚠️ Ne jamais réduire une partition montée ou utilisée par le système en cours d’exécution.

Le redimensionnement doit être effectué depuis :

- WSL avec le disque monté en mode `--bare`
- ou un Live Linux

Dans ce guide, les exemples utilisent :

```text
/dev/sdd
```

Adaptez selon votre disque.

---

# 1. Monter le disque dans WSL

Depuis PowerShell administrateur :

```powershell
Get-Disk
```

Repérer le SSD du Raspberry Pi.

Exemple :

```text
PHYSICALDRIVE3
```

Puis monter le disque dans WSL :

```powershell
wsl --mount \\.\PHYSICALDRIVE3 --bare
```

---

# 2. Vérifier les partitions

Dans WSL :

```bash
lsblk
```

Exemple :

```text
sdd
├─sdd1 512M
└─sdd2 953G
```

---

# 3. Vérifier l’intégrité du système de fichiers

Avant toute modification :

```bash
sudo e2fsck -f /dev/sdd2
```

Cette commande vérifie et corrige le système de fichiers ext4 si nécessaire.

---

# 4. Réduire le système de fichiers ext4

⚠️ Toujours réduire le filesystem AVANT la partition.

Exemple :

```bash
sudo resize2fs /dev/sdd2 55G
```

## Explication

- le système de fichiers sera réduit à 55 Go ;
- ensuite la partition sera réduite à 60 Go ;
- cela laisse une marge de sécurité de 5 Go.

⚠️ Ne jamais mettre exactement la même taille pour le filesystem et la partition.

---

# 5. Modifier les partitions avec cfdisk

Installer `cfdisk` si nécessaire :

```bash
sudo apt update
sudo apt install fdisk
```

Puis lancer :

```bash
sudo cfdisk /dev/sdd
```

---

# 6. Réduire la partition système

Dans `cfdisk` :

## Sélectionner :

```text
sdd2
```

Puis :

```text
[ Resize ]
```

Entrer :

```text
60G
```

Appuyer sur `Entrée`.

---

# 7. Créer la nouvelle partition

Vous verrez maintenant :

```text
Free Space
```

Sélectionner :

```text
[ New ]
```

Laisser la taille par défaut pour utiliser tout l’espace restant.

---

# 8. Écrire les modifications

⚠️ Les changements ne sont pas appliqués tant que vous n’avez pas validé.

Sélectionner :

```text
[ Write ]
```

Puis taper exactement :

```text
yes
```

Valider avec `Entrée`.

---

# 9. Quitter

Sélectionner :

```text
[ Quit ]
```

---

# 10. Vérifier le nouveau partitionnement

```bash
lsblk
```

Vous devriez maintenant voir :

```text
sdd
├─sdd1
├─sdd2
└─sdd3
```

---

# 11. Initialiser le chiffrement LUKS

Créer le conteneur chiffré :

```bash
sudo cryptsetup luksFormat /dev/sdd3
```

Confirmer avec :

```text
YES
```

(en majuscules)

Puis définir un mot de passe.

⚠️ Ce mot de passe sera nécessaire pour accéder aux données.

---

# 12. Ouvrir le volume chiffré

```bash
sudo cryptsetup luksOpen /dev/sdd3 ssd_data
```

Cette commande :

- déverrouille le volume ;
- crée le périphérique :

```text
/dev/mapper/ssd_data
```

---

# 13. Formater le volume déchiffré

```bash
sudo mkfs.ext4 /dev/mapper/ssd_data
```

Le chiffrement est déjà actif :
toutes les données écrites dedans seront automatiquement chiffrées.

---

# 14. Monter le volume

Créer un point de montage :

```bash
sudo mkdir /mnt/secure
```

Monter le volume :

```bash
sudo mount /dev/mapper/ssd_data /mnt/secure
```

---

# 15. Vérifier le montage

```bash
df -h
```

Vous devriez voir :

```text
/dev/mapper/ssd_data
```

monté sur :

```text
/mnt/secure
```

---

# 16. Vérifier que le chiffrement fonctionne

Vérifier l’état du volume LUKS :

```bash
sudo cryptsetup status ssd_data
```

Exemple :

```text
/dev/mapper/ssd_data is active
type: LUKS2
cipher: aes-xts-plain64
```

---

# 17. Fermer le volume chiffré

Démonter :

```bash
sudo umount /mnt/secure
```

Fermer le volume :

```bash
sudo cryptsetup luksClose ssd_data
```

Le volume redeviendra inaccessible sans le mot de passe LUKS.

---

# Résultat

Le Raspberry Pi démarre normalement sur :

```text
sdd2
```

et toutes les données sensibles peuvent être stockées de manière chiffrée dans :

```text
sdd3
```

protégé par LUKS.
