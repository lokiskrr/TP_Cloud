# TP2 : Cloud Features & Shellscripts

# I. Un p'tit nom DNS

🌞 **Prouvez que c'est effectif**

```powershell

```

# II. cloud-init

## 1. Intro

## 2. Gooooo

🌞 **Tester `cloud-init`**

```powershell

```

🌞 **Vérifier que `cloud-init` a bien fonctionné**

```powershell

```

## 3. Write your own

🌞 **Utilisez `cloud-init` pour préconfigurer une VM comme `azure2.tp2` :**

```powershell

```

🌞 **Testez que ça fonctionne**

```powershell

```

# III. Gestion de secrets

## 1. Un premier secret

🌞 **Récupérer votre secret depuis la VM**

```powershell

```

## 2. Gérer les secrets de l'application

### A. Script pour récupérer les secrets

🌞 **Coder un ptit script `bash` : `get_secrets.sh`**

```powershell

```

🌞 **Environnement du script `get_secrets.sh`**, il doit :

```powershell

```

### B. Exécution automatique

🌞 **Ajouter le script en `ExecStartPre=` dans `webapp.service`**

```powershell

```

🌞 **Prouvez que la ligne en `ExecStartPre=` a bien été exécutée**

```powershell

```

### C. Secret Flask

🌞 **Intégrez la gestion du secret Flask dans votre script `get_secrets.sh`**

```powershell

```

🌞 **Redémarrer le service**

```powershell

```

📁 **Dans le dépôt git : votre dernière version de `get_secrets.sh`**

# IV. Blob Storage

## 1. Un premier ptit blobz

🌞 **Upload un fichier dans le *Blob Container* depuis `azure2.tp2`**

```powershell

```
🌞 **Download un fichier du *Blob Container***

```powershell

```

## 2. Haïssez-moi

### A. Intro

### B. Utilisateur MySQL

🌞 **Créer un ptit user SQL pour notre script**

```powershell

```

🌞 **Tester que vous pouvez vous connecter avec cet utilisateur**

```powershell

```

### C. Script `db_backup.sh`

🌞 **Ecrire le script `db_backup.sh`**

```powershell

```
🌞 **Environnement du script `db_backup.sh`**

```powershell

```

🌞 **Récupérer le blob**

```powershell

```

📁 **Dans le dépôt git : votre `db_backup.sh`**

### D. Service

🌞 **Ecrire un fichier `/etc/systemd/system/db_backup.service`**

```powershell

```
📁 **Dans le dépôt git : votre `db_backup.service`**


🌞 **Tester**

```powershell

```

### E. Timer

🌞 **Ecrire un fichier `/etc/systemd/system/db_backup.timer`**

```powershell

```

🌞 **Activer le Timer**

```powershell

```

🌞 **Attendre et observer**

```powershell

```