# Projet Infra | MONTAGNIER Yrlan & PEYRATAUD Enzo

# Sommaire

- [TP3 : Progressons vers le réseau d'infrastructure](#tp3--progressons-vers-le-réseau-dinfrastructure)
- [Contexte du projet](#contexte-du-projet)
- [I. Logiciels & Machines utilisées](#i-outils-utilisés)
- [II. Architecture réseaux](#ii-architecture-réseaux)
  - [A. Tableau d'adressage](#a-tableau-d'adressage)
  - [B. Tableau des réseaux](#b-tableau-des-réseaux)
- [III. Technologies utilisées](#iii-technologies-utilisées)
  - [1. DHCP](#1-dhcp)
  - [2. DNS](#2-dns)
  - [3. AD-DS](#3-ad-ds)
  - [4. NAT](#4-nat)
  - [5. Partage réseau](#5-partage-réseau)
  - [6. VLAN](#6-vlan)
  - [7. Routage](#7-routage)

# Contexte du projet

- Réaliser une mini infra a l'aide de l'outil GNS3 

- Problématique :


## I. Logiciels & Machines utilisées

- GNS3 pour virtualiser une infrastructure réseau
- Virtualbox pour la **GNS3 VM** 
- VMWare pour les 3 machines : **client1**, **server1** et **admin1**
- 3 Switchs
- 1 routeur cisco (C7200)
- des VPC's

## II. Architecture réseaux

### Infrastructure sur GNS3


### A. Tableau d'adressage

| Nom du réseau | Server1.montagnier.labo | client1.montagnier.labo | VPC 1            | VPC 2            |
| ------------- | ----------------------- | :---------------------- | ---------------- | ---------------- |
| `serveurs`    | `192.168.100.2`         | `x`                     | `x`              | `x`              |
| `admins`      | `x`                     | `x`                     | `x`              | `x`              |
| `clients`     | `x`                     | `192.168.102.20`        | `192.168.102.21` | `192.168.102.22` |

### B. Tableau des réseaux

| Nom du réseau | Adresse réseau  | VLAN | Plage d'IP disponibles               | Masque                | Routeur (`R1`)  | Adresse broadcast |
| ------------- | --------------- | ---- | ------------------------------------ | --------------------- | --------------- | ----------------- |
| Serveurs      | `192.168.100.0` | 100  | `192.168.100.20` - `192.168.100.200` | `255.255.255.0` (/24) | `192.168.100.1` | `192.168.100.255` |
| Admins        | `192.168.101.0` | 101  | `192.168.101.20` - `192.168.101.200` | `255.255.255.0` (/24) | `192.168.101.1` | `192.168.101.255` |
| Clients       | `192.168.102.0` | 102  | `192.168.102.20` - `192.168.102.200` | `255.255.255.0` (/24) | `192.168.102.1` | `192.168.102.255` |

## Technologies utilisées

### 1. DHCP

- Service géré par le serveur : il va donner via le bail une ip, le nom de domaine, l'ip du routeur ainsi que celui du DNS.
- Donne IP entre 192.168.102.20 & 192.168.102.200

### 2. DNS

- Service géré par le serveur : pour la traduction des noms de domaines en IP et réciproquement.
- Domaine montagnier.labo
- Zone recherche directe
- Zone recherche indirecte

### 3. AD-DS

- Service géré par le serveur : utilisé la gestion des utilisateurs, ordinateurs, politiques de sécurité, périphérique.

#### Users and Computers

- Création et gestion des : Unités d’Organisation, utilisateurs, groupes, ordinateurs...

- Deux utilisateurs : **Enzo Peyrataud** et **Yrlan Montagnier**
  Leur mot de passe : **Client123**
  Ils accèdent au domaine à l'aide de la machine **"client1.montagnier.labo"**

- Ils sont membres du groupe **"Clients"**

- Un utilisateur qui est administrateur du domaine : **"Admin1"**
  Membre du groupe **"Admins-DSI"**
  Son mot de passe : **Admin123**
  Il accède au domaine à l'aide de la machine **"admin1.montagnier.labo"**
  
- Un routeur cisco : **R1.montagnier.labo**

### 4. Accès à Internet via node Cloud (NAT)

- Carte réseau qui va permettre l'accès à internet aux postes du 

- pour la configuration, suivre les commandes suivantes :

-

### 5. Partage réseau

- Service de partage de dossiers dans le réseau via script powershell via une politique de groupe

- Un partage pour les clients : lecture , modification, écriture , suppresion (l'admin a les privilèges pour accéder / gérer le partage des clients).

- Un partage pour les administrateurs : droit Contrôle Total (tout les droits + modifications des droits)

### 6. VLAN

- un VLAN mis en place par réseau ( un pour les clients , un pour les administrateurs et un pour le serveur )

- Seuls les administrateurs peuvent communiquer avec le VLAN du serveur.

- Les clients ne peuvent communiquer que sur leur VLAN

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
route -p ADD 192.68.100.0 MASK 255.255.255.0 192.168.102.1
route -p ADD 192.68.101.0 MASK 255.255.255.0 192.168.102.1

- sur server1 :
route -p ADD 192.68.101.0 MASK 255.255.255.0 192.168.102.1
route -p ADD 192.68.102.0 MASK 255.255.255.0 192.168.102.1

- sur admin1 :
route -p ADD 192.68.100.0 MASK 255.255.255.0 192.168.102.1
route -p ADD 192.68.102.0 MASK 255.255.255.0 192.168.102.1

### 8. Sécurisation des routeurs
