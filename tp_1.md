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

```powershell

```

### C. Agent SSH

![Encrypt SSH Keys](../../assets/img/encrypt_ssh_keys.jpg)

Afin de ne pas systématiquement saisir le mot de passe d'une clé à chaque fois qu'on l'utilise, **parce que c'est très très chiant**, on peut utiliser un **Agent SSH**.

???+ info

    Vous trouverez beaucoup d'explication sur son fonctionnement en ligne. Je recommande [**ce lien**](https://smallstep.com/blog/ssh-agent-explained/) qui est très clair sur ce qui est important à savoir.

Pour faire simple : c'est un programme que vous lancez et qui tourne ensuite en fond (commande `ssh-agent`).  
On peut lui ajouter nos clés SSH (commande `ssh-add`), qui seront ensuite être automatiquement utilisées dès qu'on fait une connexion SSH.

Les avantages sont multiples :

- **on ne saisit le mot de passe (*passphrase*) qui protège la clé qu'une seule fois** : au moment du `ssh-add`
- si la clé est dans un path chelou (genre, pas dans `~/.ssh/`) avec un nom chelou (genre `cloud_tp`), on l'ajoute une fois, **sans avoir besoin de préciser le chemin et le nom à chaque connexion**
- on peut faire suivre son agent SSH quand on se connecte à une machine A. Cela permet de faire une nouvelle connexion SSH vers une machine B, depuis la machine A, **sans avoir eu besoin de déplacer notre clé privée**

➜ **Bref, c'est juste trop pratique, et très secure.**

??? info

    Oh et y'a moyen de le faire sous tous les OS. Comme d'hab, sous Linux/MacOS, c'est moins relou qu'avec Windows :d  
    Peu importe l'OS, ça finira par taper un ptit `ssh-add` pour ajouter votre clé à l'agent normalement !

🌞 **Configurer un agent SSH sur votre poste**

- détaillez-moi toute la conf ici que vous aurez fait
- vous n'utiliserez QUE la ligne de commande, peu importe l'OS

??? note

    Au cas où j'ai besoin de le préciser : **les Windowsiens**, ça c'est obligé : vous le faites uniquement avec Powershell, votre shell natif.  
    L'agent SSH c'est natif sous Windows aussi normalement.
    
⚠️⚠️⚠️ **Tout le reste du TP DOIT être effectué avec un agent SSH actif et fonctionnel : vous ne saisissez jamais le chemin vers votre clé ou un quelconque password.**

![Logo OpenSSH](../../assets/img/logo_openssh.png)

> C'est le vrai logo de OpenSSH si vous vous demandez ce que c'est que ce truc. J'le trouve trop flex aha.
