# Mise en place des services réseau sous Debian

## Objectif

Le service DHCP (Dynamic Host Configuration Protocol) permet d’attribuer automatiquement des adresses IP ainsi que les paramètres réseau (passerelle, DNS, masque de sous-réseau) aux machines clientes.  
Cela évite une configuration manuelle sur chaque poste du réseau.

---

## Installation

Le service est installé via le gestionnaire de paquets :
    apt install isc-dhcp-server -y

### Configuration
Le fichier principal de configuration est :
    /etc/dhcp/dhcpd.conf

L’interface réseau utilisée est définie dans :
    /etc/default/isc-dhcp-server

Paramètre configuré :
INTERFACESv4="enp0s3"
### Problème rencontré

Le service ne démarrait pas initialement en raison d’une mauvaise configuration de l’interface réseau.

### Correction
Après vérification avec la commande ip a, l’interface correcte a été renseignée dans le fichier de configuration.

### Démarrage du service
    systemctl restart isc-dhcp-server
    systemctl enable isc-dhcp-server

###Résultat
Le service DHCP fonctionne correctement et attribue des adresses IP dans la plage définie.



## DNS (Bind9)
## Objectif
Le service DNS permet la résolution des noms de domaine en adresses IP et inversement.

## Installation
    apt install bind9 bind9utils -y

## Configuration

Les fichiers suivants ont été modifiés :
    /etc/bind/named.conf.options
    /etc/bind/named.conf.local
    /etc/bind/zones/

Deux zones ont été créées :
    Zone directe : mondomaine.local
    Zone inverse : 100.168.192.in-addr.arpa

### Vérification
    named-checkconf
    named-checkzone mondomaine.local /etc/bind/zones/mondomaine.local.db
    named-checkzone 100.168.192.in-addr.arpa /etc/bind/zones/192.168.100.rev

✔ Les fichiers de configuration sont valides

### Test
    dig @192.168.100.107 serveur.mondomaine.local
    dig @192.168.100.107 -x 192.168.100.107

✔ La résolution directe et inverse fonctionne correctement

## NTP (Chrony)
### Objectif

Chrony permet la synchronisation de l’heure système avec des serveurs NTP fiables.

### Installation
    apt install chrony -y

### Fonctionnement
Le serveur se synchronise avec les sources suivantes :
    time.cloudflare.com
    pool.ntp.org

La commande chronyc sources permet de vérifier les serveurs utilisés.

✔ Le serveur Cloudflare est sélectionné comme source principale.

## FIREWALL (UFW)
### Objectif
UFW (Uncomplicated Firewall) permet de filtrer les connexions réseau afin de sécuriser le serveur.

### Configuration

Politique par défaut :
    ufw default deny incoming
    ufw default allow outgoing

Règles appliquées :
    SSH (22/TCP)
    DHCP (67/UDP)
    DNS (53/TCP et UDP)
    NTP (123/UDP)

### Activation
    ufw enable
    ufw status verbose

✔ Le firewall est actif et protège le serveur


### SAMBA
##Objectif
Samba permet le partage de fichiers entre systèmes Linux et Windows via le protocole SMB.

## Installation
    apt install samba smbclient -y

## Configuration

Partage configuré :
    /srv/partage

Fichier de configuration :
    /etc/samba/smb.conf

## Utilisateur Samba

Un utilisateur système a été créé puis ajouté à Samba :
    adduser utilisateur
    smbpasswd -a utilisateur