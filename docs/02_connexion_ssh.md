# Sécurisation SSH sur Raspberry Pi 5 avec clés SSH

## Objectif

Configurer un accès SSH sécurisé sur un Raspberry Pi 5 :

- installation et vérification du serveur SSH ;
- génération d’une clé SSH ;
- connexion par clé publique ;
- désactivation des mots de passe SSH ;
- sécurisation de l’accès distant.

---

# 1. Vérifier si SSH est installé sur le Raspberry Pi

## Vérifier le statut du service SSH

```bash
sudo systemctl status ssh
```

Si SSH est actif, vous verrez :

```text
active (running)
```

---

# 2. Installer SSH si nécessaire

Si le service n’existe pas :

```bash
sudo apt update
sudo apt install openssh-server -y
```

---

# 3. Activer SSH au démarrage

```bash
sudo systemctl enable ssh
```

---

# 4. Démarrer SSH

```bash
sudo systemctl start ssh
```

---

# 5. Vérifier l’adresse IP du Raspberry Pi

```bash
hostname -I
```

Exemple :

```text
192.168.1.50
```

Cette IP servira pour la connexion SSH.

---

# 6. Vérifier si une clé SSH existe déjà sur Windows

Dans PowerShell :

```powershell
ls ~/.ssh
```

Si vous voyez :

```text
id_ed25519
id_ed25519.pub
```

alors une clé existe déjà.

---

# 7. Générer une clé SSH (recommandé)

Si vous n’avez pas encore de clé :

```powershell
ssh-keygen -t ed25519 -a 100
```

## Explication

- `ed25519` : algorithme moderne et sécurisé ;
- `-a 100` : protection renforcée contre le brute force.

---

# 8. Sauvegarder la clé

Quand cette question apparaît :

```text
Enter file in which to save the key
```

appuyer simplement sur `Entrée`.

Puis choisir une passphrase forte.

Les fichiers créés seront :

| Fichier          | Rôle                 |
| ---------------- | -------------------- |
| `id_ed25519`     | Clé privée (secrète) |
| `id_ed25519.pub` | Clé publique         |

⚠️ Ne jamais partager la clé privée.

---

# 9. Afficher la clé publique

Dans PowerShell :

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

Copier entièrement le contenu affiché.

---

# 10. Préparer le Raspberry Pi

Créer le dossier SSH :

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

---

# 11. Ajouter la clé publique

Éditer le fichier :

```bash
nano ~/.ssh/authorized_keys
```

Coller la clé publique dedans.

Sauvegarder avec :

```text
CTRL + O
```

Puis quitter :

```text
CTRL + X
```

---

# 12. Sécuriser les permissions

```bash
chmod 600 ~/.ssh/authorized_keys
```

---

# 13. Tester la connexion SSH

Depuis Windows :

```powershell
ssh utilisateur@IP_DU_RPI
```

Exemple :

```powershell
ssh utilisateur@192.168.1.50
```

Si tout fonctionne correctement :

- la connexion se fera avec la clé SSH ;
- aucun mot de passe Linux ne sera demandé.

---

# 14. Sécuriser la configuration SSH

Éditer la configuration SSH :

```bash
sudo nano /etc/ssh/sshd_config
```

Modifier ou vérifier les lignes suivantes :

```text
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin prohibit-password
```

## Explication

| Option                              | Effet                                   |
| ----------------------------------- | --------------------------------------- |
| `PubkeyAuthentication yes`          | Autorise les clés SSH                   |
| `PasswordAuthentication no`         | Désactive les mots de passe             |
| `PermitRootLogin prohibit-password` | Interdit le login root par mot de passe |

---

# 15. Redémarrer le service SSH

```bash
sudo systemctl restart ssh
```

---

# 16. Vérifier la connexion par clé

Depuis Windows :

```powershell
ssh utilisateur@IP_DU_RPI
```

Exemple :

```powershell
ssh dante@192.168.1.50
```

---

# 17. Vérifier que les mots de passe sont désactivés

Test volontaire :

```powershell
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no utilisateur@IP_DU_RPI
```

La connexion doit échouer.

---

# Résultat

Le Raspberry Pi est maintenant configuré avec :

- accès SSH sécurisé ;
- authentification par clé uniquement ;
- mots de passe SSH désactivés ;
- meilleure protection contre les attaques automatisées.
