# Projet Infra | MONTAGNIER Yrlan & PEYRATAUD Enzo
# Sommaire

- [TP3 : Progressons vers le réseau d'infrastructure](#tp3--progressons-vers-le-réseau-dinfrastructure)
- [Contexte du projet](#contexte-du-projet)
- [I. Logiciels & Machines utilisées](#i-outils-utilisés)
- [II. Architecture réseaux](#ii-architecture-réseaux)
  - [A. Tableau d'adressage](#a-tableau-d'adressage)
  - [B. Tableau des réseaux](#b-tableau-des-réseaux)
- [III. Technologies & Services utilisés/mis en place](#iii-technologies--services-utilisésmis-en-place)
  - [1. DHCP](#1-dhcp)
  - [2. DNS](#2-dns)
  - [3. AD-DS](#3-ad-ds)
  - [4. NAT](#4-nat)
  - [5. Partage réseau](#5-partage-réseau)
  - [6. VLAN](#6-vlan)
  - [7. Routage](#7-routage)
  - [8. Sécurisation des routeurs](#8-sécurisations-des-routeurs)
  - [9. IP Helper](#9-ip-helper)
- [IV. Installation](#iv-installation)

# Contexte du projet

- Réaliser une mini infra a l'aide de l'outil GNS3 
- Problématique : Simuler un cas réel d'une réalisation d'une infrastructure 

## I. Logiciels & Machines utilisées

- GNS3 pour virtualiser une infrastructure réseau
- Virtualbox pour la **GNS3 VM** 
- VMWare pour les 3 machines : **client1**, **server1** et **admin1**
- 3 Switchs
- 1 routeur cisco (C7200)
- des VPC's

## II. Architecture réseaux
### Infrastructure sur GNS3

![image](https://user-images.githubusercontent.com/71253160/171411052-64d87bd0-63ca-4eb6-9b5b-c2426d2f4364.png)

### A. Tableau d'adressage

| Nom du réseau | Server1.montagnier.labo | client1.montagnier.labo | admin1.montagnier.labo | VPC 1            | VPC 2            |
| ------------- | ----------------------- |:----------------------- | ---------------------- | ---------------- | ---------------- |
| `serveurs`    | `192.168.100.2`         | `x`                     | `x`                    | `x`              | `x`              |
| `admins`      | `x`                     | `x`                     | `192.168.101.21`       | `x`              | `x`              |
| `clients`     | `x`                     | `192.168.102.21`        | `x`                    | `192.168.102.22` | `192.168.102.23` |

### B. Tableau des réseaux

| Nom du réseau | Adresse réseau  | VLAN | Plage d'IP disponibles               | Masque                | Routeur (`R1`)  | Adresse broadcast |
| ------------- | --------------- | ---- | ------------------------------------ | --------------------- | --------------- | ----------------- |
| Serveurs      | `192.168.100.0` | 100  | `192.168.100.20` - `192.168.100.200` | `255.255.255.0` (/24) | `192.168.100.1` | `192.168.100.255` |
| Admins        | `192.168.101.0` | 101  | `192.168.101.20` - `192.168.101.200` | `255.255.255.0` (/24) | `192.168.101.1` | `192.168.101.255` |
| Clients       | `192.168.102.0` | 102  | `192.168.102.20` - `192.168.102.200` | `255.255.255.0` (/24) | `192.168.102.1` | `192.168.102.255` |

## III. Technologies & services utilisés/mis en place
### 1. DHCP

- Service installé sur le serveur : il va distribuer aux clients (Windows & VPC's) une adresse IP, le nom de domaine 'montagnier.labo', l'IP du routeur ainsi que celle du serveur DNS.

**Etendue client qui distribue :**
- Une **adresse IP** entre `192.168.102.20` & `192.168.102.200`
- **Le nom du domaine** `montagnier.labo`
- **L'IP de la passerelle** (routeur `R2`) qui est `192.168.102.1`
- **L'IP du serveur DNS** (lui même car il fait les 2) donc `192.168.100.2`
![](https://i.imgur.com/9LwXnCe.png)

- Sur les clients : 
Config IP en dhcp -> ipconfig /renew
- Sur les VPC's : 
```
PC1> ip dhcp
DORA IP 192.168.102.22/24 GW 192.168.102.1

PC2> ip dhcp
DORA IP 192.168.102.23/24 GW 192.168.102.1
```

Bail DHCP visible sur le serveur DHCP avec l'IP du client et le nom du poste dans l'étendue 192.168.101.:
![](https://i.imgur.com/q05twu0.png)

### 2. DNS
**Service installé sur le serveur : pour la traduction des noms de domaines en IP & inversement.**

#### Zones de recherche DNS

Dans le domaine `montagnier.labo`
- Zone de recherche directe

![](https://i.imgur.com/thUuNh3.png)

- Zone de recherche indirecte

![](https://i.imgur.com/C5jRPLs.png)

![](https://i.imgur.com/7fv2ULJ.png)

![](https://i.imgur.com/lVc7yzB.png)

#### Vérification du fonctionnement du DNS
- Sur le serveur :
```
PS C:\Users\Administrateur> nslookup client1
Serveur :   Server1.montagnier.labo
Address:  192.168.100.2

Nom :    client1.montagnier.labo
Address:  192.168.102.21

PS C:\Users\Administrateur> nslookup 192.168.102.21
Serveur :   Server1.montagnier.labo
Address:  192.168.100.2

Nom :    client1.montagnier.labo
Address:  192.168.102.21
```
- Sur le client : 
```
PS C:\WINDOWS\system32> nslookup server1
Serveur :   Server1.montagnier.labo
Address:  192.168.100.2

Nom :    server1.montagnier.labo
Address:  192.168.100.2
```

### 3. AD-DS

- Service géré par le serveur : utilisé la gestion des utilisateurs, ordinateurs, politiques de sécurité, périphérique.

#### Users and Computers

- Création et gestion des : Unités d’Organisation, utilisateurs, groupes, ordinateurs...

- Deux utilisateurs : **Enzo Peyrataud** et **Yrlan Montagnier**
  Leur mot de passe : **Client123**
  Ils accèdent au domaine à l'aide de la machine **"client1.montagnier.labo"**

- Ils sont membres du groupe **"Clients"**

- Un utilisateur qui est administrateur du domaine : **"admindsi" admindsi@montagnier.labo**
  Membre du groupe **"Admins-DSI"**
  Son mot de passe : **Adm123**
  Il accède au domaine à l'aide de la machine **"admin1.montagnier.labo"**
  
- Un routeur cisco : **R1.montagnier.labo**

### 4. Accès à Internet via node Cloud (NAT)

- Node `Cloud` sur GNS3 connectée à `R1`

- **Configuration de R1 : 
:file_folder: [running_config_R1](/Conf/run-conf_R1.conf)**

- **Configuration de R2 : 
:file_folder: [running_config_R2](/Conf/run-conf_R2.conf)**

- **Configuration de SW1 : 
:file_folder: [running_config_SW1](/Conf/run-conf_SW1.conf)**

- **Configuration de SW2 : 
:file_folder: [running_config_SW2](/Conf/run-conf_SW2.conf)**

- **Configuration de SW3 : 
:file_folder: [running_config_SW3](/Conf/run-conf_SW3.conf)**

### 5. Partage réseau

- Service de partage de dossiers dans le réseau via script powershell via une politique de groupe

- Un partage pour les clients : lecture , modification, écriture , suppresion (l'admin a les privilèges pour accéder / gérer le partage des clients).

- Un partage pour les administrateurs : droit Contrôle Total (tout les droits + modifications des droits)

1. On crée un partage SMB sur le Windows Server à l'emplacement `C:/Shares/Partage_Client` limité au groupe `Clients` pour que seuls eux et les admins y aient accès.
2. On met en place une GPO pour le groupe `Clients` qui programme le lancement d'un script Powershell à l'ouverture de session
3. **Voici le contenu du script :
:file_folder: Script partage réseau client : [partageClient.ps1](/Conf/partageClient.ps1)**
**:file_folder: Script partage réseau admin : [partageDSI.ps1](/Conf/partageDSI.ps1)**

### 6. VLAN

- un VLAN mis en place par réseau ( un pour les clients , un pour les administrateurs et un pour le serveur )
- Seuls les administrateurs peuvent communiquer avec le VLAN du serveur.
- Les clients ne peuvent communiquer que sur leur VLAN

**Conf des VLAN's sur les switch : 
:file_folder: [Conf_VLAN_Brief](/Conf/Conf_VLAN.txt)
:file_folder: [Conf_VLAN_Trunks](/Conf/Conf_VLAN.txt)**

### 7. Routage
- On configure notre route statique via les stratégies de groupe (GPO).
Pour cela, lancez le programme "Gestion de stratégie de groupe", puis faites un clic droit "Modifier" sur la ligne "Default Domain Policy".
- Ensuite, allez dans : Configuration ordinateur -> Stratégies -> Paramètres Windows -> Scripts (démarrage/arrêt).
Faites un double-clic sur "Démarrage".
- Cliquez sur Ajouter
- Cliquez sur "Parcourir".
- Par défaut, vous arriverez dans un dossier "Startup" vide.
Dans ce dossier, créez un fichier "add_route.bat" et collez la commande "route -p ADD ..." dans ce fichier.

- Ensuite, sélectionnez ce fichier et cliquez sur "Ouvrir" puis sur "OK", le script est ajouté.

- Redémarrez les machines du réseau local en commençant par le serveur Active Directory.
A chaque démarrage de Windows, la commande sera automatiquement exécutée.

- Si vous tapez la commande "route print" dans un invite de commandes, vous verrez que la route apparaitra dans la section "Itinéraires persistants".
Maintenant, Windows sait qu'il devra passer par le routeur (la passerelle) 192.168.x.x pour accéder aux machines du réseau 192.168.x.x

- sur client1 :
route -p ADD 192.168.100.0 MASK 255.255.255.0 192.168.102.1
route -p ADD 192.168.101.0 MASK 255.255.255.0 192.168.102.1

- sur server1 :
route -p ADD 192.168.101.0 MASK 255.255.255.0 192.168.100.1
route -p ADD 192.168.102.0 MASK 255.255.255.0 192.168.100.1

- sur admin1 :
route -p ADD 192.168.100.0 MASK 255.255.255.0 192.168.101.1
route -p ADD 192.168.102.0 MASK 255.255.255.0 192.168.101.1

Ajout route par défaut vers 0.0.0.0 0.0.0.0


### 8. Sécurisation des routeurs

### 9. IP Helper

## IV. Installation

1. Installer [GNS3](https://www.gns3.com/software/download) & [VirtualBox](https://www.virtualbox.org/wiki/Downloads) ou [VMWare](https://customerconnect.vmware.com/fr/downloads/info/slug/desktop_end_user_computing/vmware_workstation_player/16_0)
2. 
