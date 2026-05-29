# Mise en place des services réseau sous Debian

## Introduction

Dans ce projet, nous avons mis en place une machine Debian jouant le rôle de serveur central dans un réseau local. Cette infrastructure permet de simuler un environnement professionnel dans lequel un serveur gère plusieurs services essentiels au bon fonctionnement du réseau.

L’objectif est de comprendre le rôle et l’interaction de ces services dans une architecture Linux réelle, en les configurant et en les reliant entre eux.

Au fil des configurations, la machine Debian devient le cœur du réseau, capable d’automatiser la gestion des adresses IP, la résolution de noms, la synchronisation de l’heure, la sécurisation des échanges et le partage de fichiers.

---

## Objectif du projet

L’objectif principal peut se résumer en un mot : **automatisation**.

Cela concerne :
- l’attribution automatique des adresses IP  
- la résolution automatique des noms de domaine  
- la synchronisation automatique de l’heure  
- la sécurisation des accès réseau  
- le partage de ressources entre machines  

---

## Technologies utilisées

Les services mis en place sont les suivants :
- DHCP (attribution des adresses IP)
- DNS (résolution des noms)
- NTP (synchronisation de l’heure)
- UFW (pare-feu)
- Samba (partage de fichiers)


## DHCP (isc-dhcp-server - port 67/UDP)

### C’est quoi ?
Le DHCP (Dynamic Host Configuration Protocol) est un service qui attribue automatiquement une adresse IP et des paramètres réseau aux machines d’un réseau.

---

### Avec ou sans DHCP
- **Sans DHCP** → chaque machine doit être configurée manuellement (IP fixe)
- **Avec DHCP** → le serveur attribue automatiquement une IP disponible dans la plage définie

Plage configurée :
`192.168.100.150 → 192.168.100.200`

---

### Fonctionnement (DORA)
Lorsqu’un client rejoint le réseau, il suit 4 étapes :

| Étape | Action | Description |
|------|--------|-------------|
| Discover | Client → réseau | "Y a-t-il un serveur DHCP ?" |
| Offer | Serveur → client | "Je t’offre une IP" |
| Request | Client → serveur | "Je prends cette IP" |
| ACK | Serveur → client | "IP attribuée et confirmée" |


### Remarque importante
Une adresse IP DHCP n’est pas permanente : elle est **louée pour une durée limitée**.  
À expiration, le client doit renouveler sa demande.


### Configuration réalisée
- Plage d’adresses : `192.168.100.150 - 200`
- Passerelle : `192.168.100.1`
- DNS : `192.168.100.107`
- Domaine : `mondomaine.local`


## DNS (Bind9 - port 53 TCP/UDP)
### Configuration des zones

- Zone directe : `mondomaine.local` → nom → IP
- Zone inverse : `100.168.192.in-addr.arpa` → IP → nom

### Paramètres de zone
- Serial : `2026050601` → version de la zone
- Refresh : `3600` → vérification toutes les heures
- Retry : `1800` → nouvelle tentative après 30 min
- Expire : `604800` → expiration après 7 jours sans contact


### Résolution externe
- Forwarders utilisés :
  - `8.8.8.8`
  - `1.1.1.1`

Permettent de résoudre les domaines externes (Google, etc.)


## NTP - chrony (port 123/udp) 
**C'est quoi ?** Un service de synchronisation de l'heure. 

**Pourquoi c'est important ?** Beaucoup de services (logs, certificats, Kerberos, VPN...) ont besoin que toutes les machines aient la même heure. Si l'heure diffère de quelques minutes, des services peuvent refuser de fonctionner. 

**Ce qu'on a configuré :** 
- Le serveur se synchronise sur time.cloudflare.com et pool.ntp.org 
- Il est prêt à servir de référence pour les clients du réseau (allow 192.168.100.0/24) 



## **Firewall — ufw** 

**C'est quoi ?** Un pare-feu qui contrôle quel trafic est autorisé à entrer sur le serveur. Il fait allow ou deny, selon les règles. 

- 1- Paquet entrant sur port 80 

- 2- UFW vérifie ses règles dans l'ordre : 

   - Port 80 autorisé ? → OUI → le paquet passe 

   - Port 80 autorisé ? → NON → le paquet est droppé (ignoré) 


## **Politique appliquée :** 

- Tout ce qui entre est **bloqué par défaut « deny incoming»** 

- Tout ce qui sort est **autorisé « allow outgoing »** 

- On ouvre uniquement les ports nécessaires : 

|•|On ouvre|uniquement les port|
|---|---|---|
|**Port**|**Protocole**|**Service**|
|22|TCP|SSH (accès distant)|
|53|TCP + UDP|DNS|
|67|UDP|DHCP|
|123|UDP|NTP|



## **Architecture globale** 

Internet (8.8.8.8) 

[Serveur Debian] 192.168.100.107 

- `├` ── DHCP  (distribue les IPs) 

- `├` ── DNS   (résout les noms) 

`├` ── NTP   (synchronise l'heure) 

└── UFW   (firewall) 

| 

| 

[Client / WSL] reçoit tout du serveur 



## SAMBA 

Samba est un logiciel libre qui permet à un serveur Linux de partager des fichiers et des imprimantes avec des machines Windows grâce au protocole SMB. Il est utilisé pour l’interopérabilité dans les réseaux mixtes. Concrètement, il transforme mon serveur Debian en serveur de fichiers compatible Windows. 

J’ai installé Samba avec apt install samba. J’ai configuré un partage dans /etc/samba/smb.conf et créé un dossier /srv/partage. J’ai ajouté un utilisateur avec smbpasswd. Ensuite, j’ai redémarré les services smbd et nmbd. Enfin, j’ai ouvert le firewall avec ufw allow samba. Grâce à ça, ma VM Debian peut partager des fichiers avec des machines Windows ou Linux via le protocole SMB 

La commande smbpasswd -a utilisateur permet d’ajouter un utilisateur Samba. Il doit déjà exister sur le système Linux. On lui attribue un mot de passe spécifique pour Samba, qui sera utilisé par les clients Windows ou Linux pour accéder aux partages. Ensuite, je peux vérifier avec pdbedit -L que l’utilisateur est bien enregistré 

Une fois connecté avec smbclient, j’ai accès au partage Samba via le prompt smb: \>. Je peux lister les fichiers avec ls, envoyer un fichier avec put, récupérer un fichier avec get, et quitter avec exit. Cela prouve que mon partage Samba est fonctionnel et que l’utilisateur que j’ai créé peut y accéder.