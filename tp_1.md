# TP1 : Azure first steps

# I. Prérequis

## 1. Starting blocks

➜  **Activez votre compte Azure for Students**

```powershell
DONE :)
```

➜ **Installer le Azure CLI `az` sur votre poste**

```powershell
DONE :)
```

➜ **Installer Terraform sur votre poste**

```powershell
DONE :)
```

## 2. Une paire de clés SSH

### A. Choix de l'algorithme de chiffrement

🌞 **Déterminer quel algorithme de chiffrement utiliser pour vos clés**

```powershell
Il serait preferable d'utilisé ECDSA avec connexion SSH au lieu de RSA, comme recommandé dans le communiqué de l'ANSSI dans la recommandation 10 "Lorsque les clients et les serveurs SSH supportent ECDSA, son usage doit être préféré à RSA."
    -> D'apres mes recherches c'est parceque RSA est devenu obsolète et vulnérable a cause des attaques quantum computing.
https://cyber.gouv.fr/sites/default/files/2014/01/NT_OpenSSH.pdf

Mais je vais utiliser Ed25519, car il est plus rapide, stable et a une meilleur sécurité. 
```

### B. Génération de votre paire de clés

🌞 **Générer une paire de clés pour ce TP**

Key generation :
```powershell
ssh-keygen -t ed25519 -f $HOME\.ssh\cloud_tp
```

Key verification :
```powershell
PS C:\Users\makina> ls -l ~/.ssh/


    Répertoire : C:\Users\makina\.ssh


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        29/10/2025     11:58            464 cloud_tp
-a----        29/10/2025     11:58            105 cloud_tp.pub
-a----        29/10/2025     09:46           3080 known_hosts
-a----        29/10/2025     09:46           2336 known_hosts.old
```

### C. Agent SSH

🌞 **Configurer un agent SSH sur votre poste**

```powershell
PS C:\Users\makina\.ssh> Start-Service ssh-agent
PS C:\Users\makina\.ssh> Get-Service ssh-agent

Status   Name               DisplayName
------   ----               -----------
Running  ssh-agent          OpenSSH Authentication Agent

PS C:\Users\makina\.ssh> ssh-add $HOME\.ssh\cloud_tp
Enter passphrase for C:\Users\makina\.ssh\cloud_tp:
Identity added: C:\Users\makina\.ssh\cloud_tp (makina@DESKTOP-R82T9IH)
PS C:\Users\makina\.ssh> ssh-add -l
256 SHA256:Hkaf+VxcIQeVjyDgMLhvju42meW96pGQbiMa2OKTeFw makina@DESKTOP-R82T9IH (ED25519)
```

