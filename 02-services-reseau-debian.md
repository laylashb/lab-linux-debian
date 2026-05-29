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


## DHCP 
**DHCP** - **isc-dhcp-server (port 67/udp)** 

**C'est quoi ?** Un service qui distribue automatiquement des adresses IP aux machines du réseau. 

**Sans DHCP** → chaque machine doit avoir une IP configurée à la main. 

**Avec DHCP** → une machine arrive sur le réseau, elle demande une IP, le serveur lui en donne une automatiquement dans la plage 192.168.100.150 → 200. 

## **DORA pour comprendre le fonctionnement DHCP** 

Quand une machine arrive sur le réseau, il se passe 4 truc : Discover, offer, request, ack Envoi du client et des réponses du serveur : 

|-|DISCOVER  "Y'a un serveur DHCP ?"|
|---|---|
|-|OFFER|  "Oui ! Je t'ofre 192.168.100.150"|
|-|REQUEST  "Ok je la prends !"|
|-|ACK  "C'est confrmé, elle est à toi pr le moment"|



Attention, une IP DHCP n'est pas donnée pour toujours — elle est **louée** pour une durée. Après ce délai, la machine doit redemander. C'est pour éviter d'épuiser les IPs dispo. 

**Ce qu'on a configuré :** 

- Plage d'IP : 192.168.100.150 à 192.168.100.200 

- Passerelle : 192.168.100.1 

- DNS : 192.168.100.107 (notre serveur) 

- Domaine : mondomaine.local 



## **DNS BIND9** 

### **bind9 (port 53/tcp et 53/udp)** 

**C'est quoi ?** Un service qui traduit les noms en IPs (et inversement). 

**Sans DNS** → tu dois taper 192.168.100.107 pour accéder au serveur. 

**Avec DNS** → tu peux taper serveur.mondomaine.local et ça pointe vers la bonne IP. 

Serial  2026050601  → numéro de version (date + incrément) 

Refresh 3600        → les esclaves vérifient toutes les heures 

Retry   1800        → si échec, réessaie dans 30 min 

Expire  604800      → après 7 jours sans contact, la zone expire 

## **Ce qu'on a configuré :** 

- Zone directe : mondomaine.local → résout les noms en IPs 

- Zone inverse : 100.168.192.in-addr.arpa → résout les IPs en noms 

- Forwarders : 8.8.8.8 et 1.1.1.1 pour les requêtes externes (Google, YouTube, etc.) 



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