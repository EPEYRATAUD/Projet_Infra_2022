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

- **GNS3** pour **virtualiser une infrastructure réseau**
- **Virtualbox** pour la **GNS3 VM**
- **VMWare** pour les **postes Windows**
- 3 **Switchs IOU L2**
- 2 **routeurs Cisco C7200**
- Un **serveur** sous **Windows Server 2022** : `server1`
- Un **client** sous **Windows 10** : `client1`
- 2 **VPC's** qui récupèrent leur configuration IP automatiquement avec `ip dhcp`
  - `PC1` & `PC2`

## II. Architecture réseaux

### Infrastructure sur GNS3

![image](/Img/Architecture_Réseaux.png)

### A. Tableau d'adressage

| Nom du réseau | Server1.montagnier.labo | client1.montagnier.labo | admin1.montagnier.labo | VPC 1            | VPC 2            |
| ------------- | ----------------------- | :---------------------- | ---------------------- | ---------------- | ---------------- |
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

#### Configuration de l'étendue client

![Option d'étendue](/Img/Option_Etendue.png)

**L'étendue client distribue :**

- Une **adresse IP** entre `192.168.102.20` & `192.168.102.200`
- **Le nom du domaine** `montagnier.labo`
- **L'IP de la passerelle** (routeur `R2`) qui est `192.168.102.1`
- **L'IP du serveur DNS** (lui même car il fait les 2) donc `192.168.100.2`

#### Récuperer une adresse IP

- Sur le client windows :
  Config IP en dhcp -> `ipconfig /renew`
- Sur les VPC's :

```
PC1> ip dhcp
DORA IP 192.168.102.22/24 GW 192.168.102.1

PC2> ip dhcp
DORA IP 192.168.102.23/24 GW 192.168.102.1
```

Bail DHCP visible sur le serveur DHCP avec l'IP du client et le nom du poste dans l'étendue 192.168.101.:
![Bail DHCP](/Img/Bail_DHCP.png)

### 2. DNS

**Service installé sur le serveur : pour la traduction des noms de domaines en IP & inversement.**

#### Zones de recherche DNS

Dans le domaine `montagnier.labo`

- Zone de recherche directe

![Recherche directe](/Img/Recherche_Directe.png)

- Zone de recherche indirecte

![Recherche indirecte 1](/Img/Recherche_Indirecte1.png)

![Recherche indirecte 2](/Img/Recherche_Indirecte2.png)

![Recherche indirecte 3](/Img/Recherche_Indirecte3.png)

#### Vérification du fonctionnement du DNS avec nslookup

##### Résolution locale nom/ip machine

- Sur le serveur :

```
PS C:\Users\Administrateur> nslookup
Serveur par dÚfaut :   Server1.montagnier.labo
Address:  192.168.100.2

> client1
Serveur :   Server1.montagnier.labo
Address:  192.168.100.2

Nom :    client1.montagnier.labo
Address:  192.168.102.21

> 192.168.102.21
Serveur :   Server1.montagnier.labo
Address:  192.168.100.2

Nom :    client1.montagnier.labo
Address:  192.168.102.21
```

- Sur le client :

```
PS C:\Users\yrlan> nslookup server1
Serveur :   Server1.montagnier.labo
Address:  192.168.100.2

Nom :    server1.montagnier.labo
Address:  192.168.100.2
******************************************
PS C:\Users\yrlan> nslookup 192.168.102.2
Serveur :   Server1.montagnier.labo
Address:  192.168.100.2

Nom :    server1.montagnier.labo
Address:  192.168.100.2
```

##### Résolution nom de domaine extérieur (internet)

```

```

### 3. AD-DS

- Service géré par le serveur : utilisé la gestion des utilisateurs, ordinateurs, politiques de sécurité, périphérique.

#### Unité d'organisation

3 OU dans le domaine montagnier.labo :

- Ordinateurs
  - `client1.montagnier.labo`
  -
- Utilisateurs
  - Un **administrateur du domaine** :
    - AdminLabo `admLabo@montagnier.labo`
    - Mot de passe : `Admin123`
  - Deux **utilisateurs du domaine** :
    - Enzo Peyrataud `enzo@montagnier.labo`
    - Yrlan Montagnier `yrlan@montagnier.labo`
    - Mot de passe : `Client123`
  - Ils accèdent a leur session sur le domaine à l'aide de la machine **`client1.montagnier.labo`**
- Groupes
  - Admins
  - Clients
  - Serveurs

#### Ordinateurs

#### Utilisateurs

#### Groupes

- Ils sont membres du groupe **"Clients"**

- Un utilisateur qui est administrateur du domaine : **"admindsi" admindsi@montagnier.labo**
  Membre du groupe **"Admins-DSI"**
  Son mot de passe : **Adm123**
  Il accède au domaine à l'aide de la machine **"admin1.montagnier.labo"**
- Un routeur cisco : **R1.montagnier.labo**

### 4. Accès à Internet via node Cloud (NAT)

- Node `Cloud` sur GNS3 connectée à `R1`

- **Configuration de R1 :
  :file_folder: [running_config_R1](/Conf/Routeurs/run-conf_R1.conf)**

- **Configuration de R2 :
  :file_folder: [running_config_R2](/Conf/Routeurs/run-conf_R2.conf)**

- **Configuration de SW1 :
  :file_folder: [running_config_SW1](/Conf/Switchs/run-conf_SW1.conf)**

- **Configuration de SW2 :
  :file_folder: [running_config_SW2](/Conf/Switchs/run-conf_SW2.conf)**

- **Configuration de SW3 :
  :file_folder: [running_config_SW3](/Conf/Switchs/run-conf_SW3.conf)**

```
PS C:\Users\Administrateur> ping 8.8.8.8

Envoi d’une requête 'Ping'  8.8.8.8 avec 32 octets de données :
Réponse de 8.8.8.8 : octets=32 temps=131 ms TTL=113
Réponse de 8.8.8.8 : octets=32 temps=86 ms TTL=113
```

### 5. Partage réseau

- Service de partage de dossiers dans le réseau via script powershell via une politique de groupe

- Un partage pour les clients : lecture , modification, écriture , suppresion (l'admin a les privilèges pour accéder / gérer le partage des clients).

- Un partage pour les administrateurs : droit Contrôle Total (tout les droits + modifications des droits)

1. On crée un partage SMB sur le Windows Server à l'emplacement `C:/Shares/Partage_Client` limité au groupe `Clients` pour que seuls eux et les admins y aient accès.
2. Un partage réseau reservé aux admins à : `C:/Shares/Partage_DSI`
3. On met en place une GPO pour le groupe `Clients` qui programme le lancement d'un script Powershell à l'ouverture de session des clients & des admins
4. On met en place une GPO pour le groupe `Admins` qui programme le lancement d'un script Powershell à l'ouverture de session des admins
5. **Scripts pour les clients/admins :
   :file_folder: Script partage réseau client : [partageClient.ps1](/Conf/Partage/partageClient.ps1)
   :file_folder: Script partage réseau admin : [partageDSI.ps1](/Conf/Partage/partageDSI.ps1)**

### 6. VLAN

- Configuration des trunks entre les switchs & les routeurs
- Configuration des access pour les postes

**Conf des VLAN's sur les switch :
:file_folder: [Conf_VLAN_Brief](/Conf/Switchs/Conf_VLAN.txt)
:file_folder: [Conf_VLAN_Trunks](/Conf/Switchs/Conf_VLAN.txt)**

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

Ajout route par défaut vers l'IP du routeur

## IV. Installation

1. Installer [GNS3](https://www.gns3.com/software/download) & [VirtualBox](https://www.virtualbox.org/wiki/Downloads) ou [VMWare](https://customerconnect.vmware.com/fr/downloads/info/slug/desktop_end_user_computing/vmware_workstation_player/16_0)
2. Récuperer les appliances des routeurs C7200 et Switch cisco IOU_L2
3. Configurer les routeurs et switchs en suivant les fichiers de configuration fournis
4. Installer un serveur Windows avec AD-DS, DHCP, DNS, Partage de fichiers
5. Installation de Windows 10 sur une VM puis Windows Server 2022 sur une seconde
6. Intégration au domaine AD, obtention d'un bail DHCP sur le client
