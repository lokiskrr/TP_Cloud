# TP2 : Cloud Features & Shellscripts

# I. Un p'tit nom DNS

🌞 **Prouvez que c'est effectif**

```powershell
PS C:\Users\makina> az network public-ip update --resource-group cat --name azure1.tp1PublicIP --dns-name meow-azure1
{
  "ddosSettings": {
    "protectionMode": "VirtualNetworkInherited"
  },
  "dnsSettings": {
    "domainNameLabel": "meow-azure1",
    "fqdn": "meow-azure1.swedencentral.cloudapp.azure.com"
  },
  "etag": "W/\"23fd66ad-31a8-4519-b07b-a60b1fb43c90\"",
  "id": "/subscriptions/eda0215f-297b-4358-90a3-9da218cca3ca/resourceGroups/cat/providers/Microsoft.Network/publicIPAddresses/azure1.tp1PublicIP",
  "idleTimeoutInMinutes": 4,
  "ipAddress": "20.91.250.120",
  "ipConfiguration": {
    "id": "/subscriptions/eda0215f-297b-4358-90a3-9da218cca3ca/resourceGroups/cat/providers/Microsoft.Network/networkInterfaces/azure1.tp1VMNic/ipConfigurations/ipconfigazure1.tp1",
    "resourceGroup": "cat"
  },
  "ipTags": [],
  "location": "swedencentral",
  "name": "azure1.tp1PublicIP",
  "provisioningState": "Succeeded",
  "publicIPAddressVersion": "IPv4",
  "publicIPAllocationMethod": "Static",
  "resourceGroup": "cat",
  "resourceGuid": "defc390e-f329-4f10-acb5-113d05d542ea",
  "sku": {
    "name": "Standard",
    "tier": "Regional"
  },
  "tags": {},
  "type": "Microsoft.Network/publicIPAddresses"
}
```

Differentes vérification : 
```powershell 
PS C:\Users\makina> az vm list -d -g cat --query "[].{VMName:name, IP:publicIps}" -o table
VMName      IP
----------  --------------
azur1       135.225.33.222
azure1.tp1  20.91.250.120
azure2.tp1

PS C:\Users\makina> az network public-ip show -g cat -n azure1.tp1PublicIP --query "{DNS: dnsSettings.fqdn, IP: ipAddress}" -o table
DNS                                           IP
--------------------------------------------  -------------
meow-azure1.swedencentral.cloudapp.azure.com  20.91.250.120
```

```powershell 
PS C:\Users\makina> curl http://meow-azure1.swedencentral.cloudapp.azure.com:8000


StatusCode        : 200
StatusDescription : OK
Content           : <!DOCTYPE html>
                    <html lang="en">
                    <head>
                        <meta charset="UTF-8">
                        <meta name="viewport" content="width=device-width, initial-scale=1.0">
                        <title>Purr Messages - Cat Message Board</title>
```
# II. cloud-init

## 1. Intro

## 2. Gooooo

🌞 **Tester `cloud-init`**

```powershell
PS C:\Users\makina\.ssh> az vm create 
--resource-group cat 
--name azure3.tp2 
--image Ubuntu2404 
--size Standard_B1s 
--ssh-key-values ~/.ssh/cloud_tp.pub 
--custom-data ~/.ssh/cloud-init.txt 
--public-ip-sku Standard 
--location swedencentral
The default value of '--size' will be changed to 'Standard_D2s_v5' from 'Standard_DS1_v2' in a future release.
{
  "fqdns": "",
  "id": "/subscriptions/eda0215f-297b-4358-90a3-9da218cca3ca/resourceGroups/cat/providers/Microsoft.Compute/virtualMachines/azure3.tp2",
  "location": "swedencentral",
  "macAddress": "7C-ED-8D-9D-F6-38",
  "powerState": "VM running",
  "privateIpAddress": "172.17.0.7",
  "publicIpAddress": "4.223.117.53",
  "resourceGroup": "cat"
}
```
```powershell
 ssh fata@4.223.117.53
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.14.0-1012-azure x86_64)
[...]

PS C:\Users\makina\.ssh> ssh fata@4.223.117.53
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.14.0-1012-azure x86_64)
[...]
```
🌞 **Vérifier que `cloud-init` a bien fonctionné**

```powershell
fata@azure3:~$ sudo systemctl status cloud-init
● cloud-init.service - Cloud-init: Network Stage
     Loaded: loaded (/usr/lib/systemd/system/cloud-init.service; enabled; preset: enabled)
     Active: active (exited) since Thu 2025-10-30 16:51:14 UTC; 9min ago

fata@azure3:~$ cloud-init status
status: done
```
```powershell 
fata2@azure3:~$ sudo systemctl status cloud-init
● cloud-init.service - Cloud-init: Network Stage
     Loaded: loaded (/usr/lib/systemd/system/cloud-init.service; enabled; preset: enabled)
     Active: active (exited) since Thu 2025-10-30 16:51:14 UTC; 10min ago
   Main PID: 705 (code=exited, status=0/SUCCESS)
        CPU: 1.528s

fata2@azure3:~$ cloud-init status
status: done
```
## 3. Write your own

🌞 **Utilisez `cloud-init` pour préconfigurer une VM comme `azure2.tp2` :**

Cloud-init -->
```powershel 
#cloud-config
disable_root: false

system_info:
  default_user:
    name: fata

users:
  - name: fata
    sudo: ALL=(ALL) NOPASSWD:ALL
    passwd: $6$kkzNH2tCq4l341MH$3mVCwbmU7.PMKGkZTeU8ock66zM4sduqUiRZDaU2df/hN0L5tDSe5W2GR9OVL.OzDPeQgg4r4/QxKOJErsgoO.
    groups: [sudo]
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAEEacSrl3NVn5qCat8ucBMqEfBHR7274Ttubg+ALNDF makina@DESKTOP-R82T9IH

packages:
  - mariadb-server

write_files:
  - path: /tmp/init.sql
    owner: root:root
    permissions: '0600'
    content: |
      CREATE DATABASE IF NOT EXISTS meow_database;
      CREATE USER IF NOT EXISTS 'meow'@'%' IDENTIFIED BY 'meow';
      GRANT ALL PRIVILEGES ON meow_database.* TO 'meow'@'%';
      FLUSH PRIVILEGES;

runcmd:
  - systemctl enable mariadb
  - systemctl start mariadb
  - mariadb -u root < /tmp/init.sql
#  - rm -f /tmp/init.sql
```

🌞 **Testez que ça fonctionne**

```powershell
fata@meown:~$ mysql -u meow -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 35
Server version: 10.6.22-MariaDB-0ubuntu0.22.04.1 Ubuntu 22.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| meow_database      |
+--------------------+
2 rows in set (0.000 sec)
```

# III. Gestion de secrets

## 1. Un premier secret

🌞 **Récupérer votre secret depuis la VM**

keyvault creation -->
```powershell
az>> keyvault create --name fatakv --resource-group cat --location swedencentral --enable-rbac-authorization false
```

secret creation --> 
```powershell 
az>> keyvault secret set --vault-name fatakv --name fatasecret --val
```

manage idendity activation -->
```powershell 
az>> vm identity assign --name azure1.tp1 --resource-group cat
```

access policy configuration -->
```powershell 
az keyvault set-policy --name fatakv --object-id c7c8f8d9-a64d-4a09-8da4-d2d247668205 --secret-permissions get list
```

```powershell 
az login --identity --allow-no-subscriptions

azureuser@azure1:~$ az keyvault secret show --vault-name fatakv --name TestSecret
{
  "attributes": {
    "created": "2025-10-31T10:40:56+00:00",
    "enabled": true,
    "expires": null,
    "notBefore": null,
    "recoverableDays": 90,
    "recoveryLevel": "Recoverable+Purgeable",
    "updated": "2025-10-31T10:40:56+00:00"
  },
  "contentType": null,
  "id": "https://fatakv.vault.azure.net/secrets/TestSecret/d307c56bac994d2e922effa962d4b595",
  "kid": null,
  "managed": null,
  "name": "TestSecret",
  "tags": {
    "file-encoding": "utf-8"
  },
  "value": "ValeurDeTest123"
}
```
## 2. Gérer les secrets de l'application

### A. Script pour récupérer les secrets

🌞 **Coder un ptit script `bash` : `get_secrets.sh`**

new secret -->
```powershell
az>> keyvault secret set --vault-name fatakv --name DBPASSWORD --value "meow"
```

script bash : get-secrets.sh
```powershell 
az login --identity --allow-no-subscriptions >/dev/null 2>&1

SECRET=$(az keyvault secret show --vault-name fatakv --name DBPASSWORD --query value -o tsv)

ENV_PATH="/home/webapp/app/.env"

mkdir -p "$(dirname "$ENV_PATH")"
touch "$ENV_PATH"

grep -q '^DB_PASSWORD=' "$ENV_PATH" && \
    sed -i "s/^DB_PASSWORD=.*/DB_PASSWORD=$SECRET/" "$ENV_PATH" || \
    echo "DB_PASSWORD=$SECRET" >> "$ENV_PATH"

echo "DB_PASSWORD mis à jour dans $ENV_PATH"
```

🌞 **Environnement du script `get_secrets.sh`**, il doit :

```powershell
azureuser@azure1:~$ sudo mv ./get_secrets.sh /usr/local/bin/get_secrets.sh

sudo chown webapp:webapp /usr/local/bin/get_secrets.sh

sudo chmod 700 /usr/local/bin/get_secrets.sh

azureuser@azure1:/usr/local/bin$ ls -l /usr/local/bin/get_secrets.sh

azureuser@azure1:/usr/local/bin$ sudo -u webapp /usr/local/bin/get_secrets.sh
DB_PASSWORD mis à jour dans /home/webapp/app/.env
```

### B. Exécution automatique

🌞 **Ajouter le script en `ExecStartPre=` dans `webapp.service`**

```powershell
azureuser@azure1:/usr/local/bin$ sudo nano /etc/systemd/system/webapp.service

azureuser@azure1:/$ sudo systemctl daemon-reload

azureuser@azure1:/$ sudo systemctl restart webapp
```

🌞 **Prouvez que la ligne en `ExecStartPre=` a bien été exécutée**

```powershell
azureuser@azure1:/$ systemctl status webapp
● webapp.service - Super Webapp MEOW
     Loaded: loaded (/etc/systemd/system/webapp.service; enable>
     Active: active (running) since Fri 2025-10-31 11:45:38 UTC>
    Process: 19683 ExecStartPre=/usr/local/bin/get_secrets.sh (>
   Main PID: 19737 (python)
      Tasks: 1 (limit: 989)
     Memory: 53.5M (peak: 55.6M)
        CPU: 1.870s
     CGroup: /system.slice/webapp.service
             └─19737 /opt/meow/bin/python app.py
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