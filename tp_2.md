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
  - path: /home/fata/init.sql
    owner: fata:fata
    permissions: '0644'
    content: |
      CREATE DATABASE IF NOT EXISTS meow_database;
      CREATE USER IF NOT EXISTS 'meow'@'%' IDENTIFIED BY 'meow';
      GRANT ALL PRIVILEGES ON meow_database.* TO 'meow'@'%';
      FLUSH PRIVILEGES;

runcmd:
  - sudo systemctl enable mariadb
  - sudo systemctl start mariadb
  - sudo mariadb -u root < /home/fata/init.sql
```

```powershell
PS C:\Users\makina> az vm create `
>> --resource-group cat `
>> --name azure5.tp2 `
>> --image Ubuntu2404 `
>> --size Standard_B1s `
>> --ssh-key-values ~/.ssh/cloud_tp.pub `
>> --custom-data ~/.ssh/cloud-init.txt `
>> --public-ip-sku Standard `
>> --location SwedenCentral
The default value of '--size' will be changed to 'Standard_D2s_v5' from 'Standard_DS1_v2' in a future release.
{
  "fqdns": "",
  "id": "/subscriptions/eda0215f-297b-4358-90a3-9da218cca3ca/resourceGroups/cat/providers/Microsoft.Compute/virtualMachines/azure5.tp2",
  "location": "swedencentral",
  "macAddress": "7C-1E-52-1D-22-E0",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "4.223.141.211",
  "resourceGroup": "cat"
}
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