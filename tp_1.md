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
```

# II. Spawn des VMs

## 1. Depuis la WebUI


🌞 **Connectez-vous en SSH à la VM pour preuve**

```powershell
PS C:\Users\makina> ssh azureuser@135.225.33.222
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.14.0-1012-azure x86_64)
[...]
azureuser@azur1:~$
```

## 2. `az` : a programmatic approach

🌞 **Créez une VM depuis le Azure CLI** : `azure1.tp1`

```powershell
PS C:\Users\makina> az vm create `
>> --resource-group cat `
>> --name azure1.tp1 `
>> --image Ubuntu2404 `
>> --size Standard_B1s `
>> --admin-username azureuser `
>> --ssh-key-values "C:\Users\makina\.ssh\cloud_tp.pub"`
>> --public-ip-sku Standard `
>> --location swedencentral `
>>
The default value of '--size' will be changed to 'Standard_D2s_v5' from 'Standard_DS1_v2' in a future release.
{
  "fqdns": "",
  "id": "/subscriptions/eda0215f-297b-4358-90a3-9da218cca3ca/resourceGroups/cat/providers/Microsoft.Compute/virtualMachines/azure1.tp1",
  "location": "swedencentral",
  "macAddress": "7C-1E-52-3F-5C-9D",
  "powerState": "VM running",
  "privateIpAddress": "172.17.0.5",
  "publicIpAddress": "20.91.250.120",
  "resourceGroup": "cat"
}
```
🌞 **Assurez-vous que vous pouvez vous connecter à la VM en SSH sur son IP publique**

```powershell
PS C:\Users\makina> ssh azureuser@20.91.250.120
[...]
azureuser@cloud:~$
```

🌞 **Une fois connecté, prouvez la présence...**

```powershell
azureuser@azure1:~$ sudo systemctl status walinuxagent
● walinuxagent.service - Azure Linux Agent
     Loaded: loaded (/usr/lib/systemd/system/walinuxagent.service; enabled; preset: enabled)
    Drop-In: /run/systemd/system.control/walinuxagent.service.d
             └─50-CPUAccounting.conf, 50-MemoryAccounting.conf
     Active: active (running) since Thu 2025-10-30 09:55:45 UTC; 2min 2s ago
```
```powershell
azureuser@azure1:~$ sudo systemctl status cloud-init
● cloud-init.service - Cloud-init: Network Stage
     Loaded: loaded (/usr/lib/systemd/system/cloud-init.service; enabled; preset: enabled)
     Active: active (exited) since Thu 2025-10-30 09:55:45 UTC; 2min 26s ago
   Main PID: 704 (code=exited, status=0/SUCCESS)
        CPU: 1.606s
```

## 3. Spawn moar moar moaaar VMs
### A. Another VM another friend :d

🌞 **Créez une deuxième VM** : `azure2.tp1`

```powershell
PS C:\Users\makina> az vm create `
>> --resource-group cat `
>> --name azure2.tp1 `
>> --public-ip-address '""'`
>> --image Ubuntu2404 `
>> --size Standard_B1s `
>> --admin-username azureuser `
>> --ssh-key-values "C:\Users\makina\.ssh\cloud_tp.pub"`
>> --public-ip-sku Standard `
>> --location swedencentral `
The default value of '--size' will be changed to 'Standard_D2s_v5' from 'Standard_DS1_v2' in a future release.
{
  "fqdns": "",
  "id": "/subscriptions/eda0215f-297b-4358-90a3-9da218cca3ca/resourceGroups/cat/providers/Microsoft.Compute/virtualMachines/azure2.tp1",
  "location": "swedencentral",
  "macAddress": "60-45-BD-FB-A7-08",
  "powerState": "VM running",
  "privateIpAddress": "172.17.0.6",
  "publicIpAddress": "",
  "resourceGroup": "cat"
}
```
🌞 **Affichez des infos au sujet de vos deux VMs**

```powershell
PS C:\Users\makina> az vm list-ip-addresses --output table
VirtualMachine    PublicIPAddresses    PrivateIPAddresses
----------------  -------------------  --------------------
azur1             135.225.33.222       172.17.0.4
azure1.tp1        20.91.250.120        172.17.0.5
azure2.tp1                             172.17.0.6
```

### B. Config SSH client

🌞 **Configuration SSH client pour les deux machines**

```powershell
PS C:\Users\makina> notepad $HOME/.ssh/config
```

```powershell
Host az1
    HostName 135.225.92.52
    User azureuser
    IdentityFile C:\Users\makina\.ssh\cloud_tp
    IdentitiesOnly yes

Host az2
    HostName 172.17.0.6
    User azureuser
    IdentityFile C:\Users\makina\.ssh\cloud_tp
    ProxyJump az1
    IdentitiesOnly yes
```

```powershell 
PS C:\Users\makina\.ssh> ssh az1
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.14.0-1012-azure x86_64)
[...]
azureuser@azure1:~$ exit
logout
Connection to 20.91.250.120 closed.
```

```powershell
PS C:\Users\makina> ssh az2
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.14.0-1012-azure x86_64)
[...]
azureuser@azure2:~$
```

# III. Déployer et configurer un machin

## 1. Machine `azure2.tp1`

🌞 **Installer MySQL/MariaDB sur `azure2.tp1`**

```powershell
azureuser@azure2:~$ sudo apt install mariadb-server -y
```

🌞 **Démarrer le service MySQL/MariaDB sur `azure2.tp1`**

```powershell
azureuser@azure2:~$ sudo systemctl start mariadb
```

🌞 **Ajouter un utilisateur dans la base de données pour que mon app puisse s'y connecter**

```powershell
sudo mariadb
```

```powershell
CREATE DATABASE meow_database;
CREATE USER 'meow'@'%' IDENTIFIED BY 'meow';
GRANT ALL ON meow_database.* TO 'meow'@'%';
FLUSH PRIVILEGES;
```

🌞 **Ouvrez un port firewall si nécessaire**

```powershell
azureuser@azure2:~$ sudo ss -lnpt
azureuser@azure2:/etc/mysql$ sudo ss -lnpt
State    Recv-Q   Send-Q     Local Address:Port       Peer Address:Port   Process
LISTEN   0        4096       127.0.0.53%lo:53              0.0.0.0:*       users:(("systemd-resolve",pid=486,fd=15))
LISTEN   0        4096          127.0.0.54:53              0.0.0.0:*       users:(("systemd-resolve",pid=486,fd=17))
LISTEN   0        80               0.0.0.0:3306            0.0.0.0:*       users:(("mariadbd",pid=3796,fd=24))
LISTEN   0        4096             0.0.0.0:22              0.0.0.0:*       users:(("sshd",pid=1658,fd=3),("systemd",pid=1,fd=201))
LISTEN   0        4096                [::]:22                 [::]:*       users:(("sshd",pid=1658,fd=4),("systemd",pid=1,fd=202))
```

## 2. Machine `azure1.tp1`

### A. Récupération de l'application sur la machine

🌞 **Récupération de l'application sur la machine**

```powershell
azureuser@azure1:~$ sudo mkdir -p /opt/meow
azureuser@azure1:~$ sudo chown $USER:$USER /opt/meow
azureuser@azure1:~$ cd /opt/meow
azureuser@azure1:/opt/meow$ git clone https://gitlab.com/it4lik/b2-pano-cloud-2025.git .
Cloning into '.'...
remote: Enumerating objects: 399, done.
remote: Counting objects: 100% (319/319), done.
remote: Compressing objects: 100% (316/316), done.
remote: Total 399 (delta 150), reused 0 (delta 0), pack-reused 80 (from 1)
Receiving objects: 100% (399/399), 14.26 MiB | 16.06 MiB/s, done.
Resolving deltas: 100% (173/173), done.
```

```powershell
azureuser@azure1:/opt/meow$ cd docs/tp/1/app
azureuser@azure1:/opt/meow/docs/tp/1/app$ ls
app.py  docker-compose.yml  requirements.txt  templates
azureuser@azure1:/opt/meow/docs/tp/1/app$
```

### B. Installation des dépendances de l'application

🌞 **Installation des dépendances de l'application**

```powershell
azureuser@azure1:/opt/meow/docs/tp/1/app$ python3 -m venv .
azureuser@azure1:/opt/meow/docs/tp/1/app$ /opt/meow/bin/pip install -r requirements.txt

```
### C. Configuration de l'application

🌞 **Configuration de l'application**

```powershell
azureuser@azure1:/opt/meow nano .env
```

```powershell
DB_HOST=localhost=172.17.0.6
```
### D. Gestion de users et de droits

🌞 **Gestion de users et de droits**

```powershell
azureuser@azure1:/opt/meow/docs/tp/1/app$ sudo useradd -m -s /bin/bash webapp
azureuser@azure1:/opt/meow/docs/tp/1/app$ sudo passwd webapp
New password:
Retype new password:
passwd: password updated successfully
```

```powershell
azureuser@azure1:~$ sudo chown -R webapp:webapp /opt/meow
azureuser@azure1:/opt/meow/docs/tp/1/app$ sudo chmod -R o-rwx /opt/meow
```

### E. Création d'un service `webapp.service` pour lancer l'application

🌞 **Création d'un service `webapp.service` pour lancer l'application**

```powershell
[Unit]
Description=Super Webapp MEOW

[Service]
User=webapp
WorkingDirectory=/opt/meow
ExecStart=/opt/meow/bin/python app.py

[Install]
WantedBy=multi-user.target
```

```powershell
azureuser@azure1:~$ sudo systemctl daemon-reload
azureuser@azure1:~$ sudo systemctl enable webapp
azureuser@azure1:~$ sudo systemctl start webapp

azureuser@azure1:~$ sudo systemctl status webapp
● webapp.service - Super Webapp MEOW
     Loaded: loaded (/etc/systemd/system/webapp.service; enable>
     Active: active (running) since Thu 2025-10-30 14:29:36 UTC>
   Main PID: 6233 (python)
      Tasks: 1 (limit: 989)
     Memory: 41.3M (peak: 43.3M)
        CPU: 499ms
     CGroup: /system.slice/webapp.service
             └─6233 /opt/meow/bin/python app.py
```
### F. Ouverture du port dans le(s) firewall(s)

🌞 **Ouverture du port80 dans le(s) firewall(s)**

```powershell
azureuser@azure1:~$ sudo ss -lnpt
State    Recv-Q   Send-Q      Local Address:Port       Peer Address:Port   Process
LISTEN   0        4096              0.0.0.0:22              0.0.0.0:*       users:(("sshd",pid=1577,fd=3),("systemd",pid=1,fd=97))
LISTEN   0        4096           127.0.0.54:53              0.0.0.0:*       users:(("systemd-resolve",pid=508,fd=17))
LISTEN   0        128               0.0.0.0:8000            0.0.0.0:*       users:(("python",pid=6233,fd=4))
LISTEN   0        4096        127.0.0.53%lo:53              0.0.0.0:*       users:(("systemd-resolve",pid=508,fd=15))
LISTEN   0        4096                 [::]:22                 [::]:*       users:(("sshd",pid=1577,fd=4),("systemd",pid=1,fd=98))
```

```powershell
azureuser@azure1:~$ sudo ufw allow 80/tcp
Rules updated
Rules updated (v6)
```
## 3. Visitez l'application

🌞 **L'application devrait être fonctionnelle sans soucis à partir de là**

```powershell
PS C:\Users\leobe> curl http://20.91.250.120:8000/


StatusCode        : 200
StatusDescription : OK
Content           : <!DOCTYPE html>
                    <html lang="en">
                    <head>
                        <meta charset="UTF-8">
                        <meta name="viewport"
                    content="width=device-width, initial-scale=1.0">
                        <title>Purr Messages - Cat Message
                    Board</title>
                        <...
```

