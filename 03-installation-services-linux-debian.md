## DHCP 

## **Objectif** 

Le DHCP (Dynamic Host Configuration Protocol) permet d’attribuer automatiquement une adresse IP et les paramètres réseau (masque, passerelle, DNS) aux machines du réseau. Sans lui, chaque poste devrait être configuré manuellement. 

## **Étapes** 

Connexion root : 

user1@debian13-server:~$ su - 

Mot de passe : 

root@debian13-server:~# 

Je passe en super-utilisateur pour avoir les droits nécessaires. 

Mise à jour et installation : 

root@debian13-server:~# sudo apt update && sudo apt upgrade -y 

root@debian13-server:~# sudo apt install isc-dhcp-server -y 

Mise à jour du système et installation du service DHCP. 

Commande : mv /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.bak 

Vérification de l’interface : 

root@debian13-server:~# ip a show enp0s3 

Cette commande me permet de confirmer le nom de l’interface réseau et son adresse IP. Ici, c’est enp0s3. 

Configuration : 

root@debian13-server:~# sudo nano /etc/dhcp/dhcpd.conf 

J’indique l’interface utilisée par DHCP. 

****** Erreur rencontrée : le service ne démarrait pas. 

Correction dans : 

root@debian13-server:~# nano /etc/default/isc-dhcp-server 

INTERFACESv4="enp0s3" 

Le problème venait du mauvais fichier, il fallait modifier /etc/default/isc-dhcp-server. 

Démarrage : 

root@debian13-server:~# systemctl restart isc-dhcp-server 

root@debian13-server:~# systemctl enable isc-dhcp-server 

isc-dhcp-server.service is not a native service, redirecting to systemd-sysv-install. 

Executing: /usr/lib/systemd/systemd-sysv-install enable isc-dhcp-server 

root@debian13-server:~# systemctl status isc-dhcp-server 

Le message indique que le service est géré via SysV, mais il est bien activé. 

## **2. Bind9 (DNS)** 

## **Objectif** 

Bind9 est un serveur DNS. Il traduit les noms de domaine en adresses IP (résolution directe) et inversement (résolution inverse). C’est indispensable pour que les machines du réseau puissent se joindre par nom. 

## **Étapes** 

Installation : 

root@debian13-server:~# apt install bind9 bind9utils -y 

J’installe Bind9 et ses utilitaires. Le système crée un lien vers named.service. 

Configuration des options : 

root@debian13-server:~# nano /etc/bind/named.conf.options 

Je définis le répertoire de cache, les clients autorisés, l’adresse IP d’écoute, et les DNS externes (Google et Cloudflare) en cas de besoin. 

Création des zones : 

root@debian13-server:~# nano /etc/bind/named.conf.local 

root@debian13-server:~# mkdir /etc/bind/zones 

root@debian13-server:~# nano /etc/bind/zones/mondomaine.local.db 

root@debian13-server:~# nano /etc/bind/zones/192.168.100.rev 

Je crée les fichiers de zone directe et inverse pour mon domaine mondomaine.local. 

Vérification : 

root@debian13-server:~# named-checkconf 

root@debian13-server:~# named-checkzone mondomaine.local 

/etc/bind/zones/mondomaine.local.db 

zone mondomaine.local/IN: loaded serial 2026050601 

## OK 

root@debian13-server:~# named-checkzone 100.168.192.in-addr.arpa 

/etc/bind/zones/192.168.100.rev 

zone 100.168.192.in-addr.arpa/IN: loaded serial 2026050601 

## OK 

Les fichiers de zone sont valides. 

Démarrage : 

root@debian13-server:~# systemctl restart bind9 

root@debian13-server:~# systemctl enable bind9 

Failed to enable unit: Refusing to operate on linked unit file bind9.service root@debian13-server:~# systemctl enable named 

root@debian13-server:~# systemctl status bind9 

Erreur corrigée en activant named au lieu de bind9. 

## Tests : 

root@debian13-server:~# dig @192.168.100.107 serveur.mondomaine.local 

Résultat : 

serveur.mondomaine.local. 604800 IN A 192.168.100.107 

Résolution directe OK. 

root@debian13-server:~# dig @192.168.100.107 -x 192.168.100.107 

Résultat : 

107.100.168.192.in-addr.arpa. 604800 IN PTR serveur.mondomaine.local. 

Résolution inverse OK. 

## **3. Chrony (NTP)** 

## **Objectif** 

Chrony est un service NTP qui synchronise l’heure du serveur avec des sources fiables. Une bonne synchronisation est essentielle pour les logs, la sécurité et le bon fonctionnement des services. 

## **Étapes** 

Installation : 

root@debian13-server:~# apt install chrony -y 

Configuration et démarrage : root@debian13-server:~# nano /etc/chrony/chrony.conf root@debian13-server:~# systemctl restart chrony root@debian13-server:~# systemctl enable chrony root@debian13-server:~# systemctl status chrony 

Résultat : chronyd version 4.6.1 starting Selected source time.cloudflare.com System clock TAI offset set to 37 seconds Vérification : root@debian13-server:~# chronyc sources 

MS Name/IP address         Stratum Poll Reach LastRx Last sample 

layla@debian13-server:~$ **dig serveur.local** 

^Clayla@debian13-server:~$ **dig serveur.local** 

^Clayla@debian13-server:~$ **timedatectl status** Local time: jeu. 2026-05-07 14:44:12 CEST Universal time: jeu. 2026-05-07 12:44:12 UTC RTC time: jeu. 2026-05-07 12:44:13 Time zone: Europe/Paris (CEST, +0200) System clock synchronized: no 

NTP service: active 

RTC in local TZ: no 

## layla@debian13-server:~$ **timedatectl set-ntp true** 

==== AUTHENTICATING FOR org.freedesktop.timedate1.set-ntp ==== 

Une authentification est requise pour activer ou désactiver la synchronisation de l'heure avec le réseau. 

Multiple identities can be used for authentication: 

1.  layla,,, (layla) 

2.  ,,, (adminuser) 

3.  admin 

Choose identity to authenticate as (1-3): 1 

Password: 

==== AUTHENTICATION COMPLETE ==== 

layla@debian13-server:~$ 

====================================================================== 

========= 

^- ntp.marwan.ma                 2   6    27    42    -47ms[  -47ms] +/-  154ms 

^* time.cloudflare.com           3   6    17    45   -175us[  -13ms] +/-   27ms 

^- ip217-154-21-219.pbiaas.>     2   6    17    45  +4224us[+4224us] +/-   39ms 

Chrony synchronise avec plusieurs serveurs. L’étoile ^* indique la source active (ici Cloudflare). 

## **4. UFW (Firewall)** 

## **Objectif** 

UFW (Uncomplicated Firewall) permet de sécuriser le serveur en bloquant par défaut les connexions entrantes, sauf celles explicitement autorisées. 

## **Étapes** 

Installation : 

root@debian13-server:~# apt install ufw -y 

Politique par défaut : 

root@debian13-server:~# ufw default deny incoming 

root@debian13-server:~# ufw default allow outgoing 

Je bloque tout en entrée par défaut, et j’autorise tout en sortie. 

Règles : 

root@debian13-server:~# ufw allow ssh 

root@debian13-server:~# ufw allow 67/udp root@debian13-server:~# ufw allow 53/tcp 

root@debian13-server:~# ufw allow 53/udp 

root@debian13-server:~# ufw allow 123/udp 

J’autorise uniquement les services nécessaires : SSH, DHCP, DNS et NTP. 

Activation : 

root@debian13-server:~# ufw enable 

root@debian13-server:~# ufw status verbose 

Résultat : 

Status: active 

22/tcp   ALLOW IN    Anywhere 

67/udp   ALLOW IN    Anywhere 53/tcp   ALLOW IN    Anywhere 

## 53/udp   ALLOW IN    Anywhere 

123/udp  ALLOW IN    Anywhere 

Le firewall est actif et protège le serveur. 

## Samba 

admin@debian13-server:~$ systemctl status smbd 

- smbd.service - Samba SMB Daemon 

Loaded: loaded (/usr/lib/systemd/system/smbd.service; enabled; preset: enabled) 

Active: active (running) since Wed 2026-05-06 21:43:41 CEST; 12s ago Invocation: 3c19eea81f7a41828cd7faab7e3624fa 

Docs: man:smbd(8) man:samba(7) man:smb.conf(5) Main PID: 12630 (smbd) Status: "smbd: ready to serve connections..." Tasks: 3 (limit: 2301) Memory: 10.1M (peak: 10.3M) 

CPU: 483ms 

CGroup: /system.slice/smbd.service 

`├` ─12630 /usr/sbin/smbd --foreground --no-process-group 

`├` ─12635 "smbd: notifyd" . └─12636 "smbd: cleanupd " 

admin@debian13-server:~$ systemctl status nmbd 

● nmbd.service - Samba NMB Daemon 

Loaded: loaded (/usr/lib/systemd/system/nmbd.service; enabled; preset: enabled) Active: active (running) since Wed 2026-05-06 21:43:42 CEST; 19s ago 

Invocation: 8a5d2bccaf404a18a3d4b7c2cb3f1b41 

Docs: man:nmbd(8) 

man:samba(7) 

man:smb.conf(5) Main PID: 12680 (nmbd) 

Status: "nmbd: ready to serve connections..." 

Tasks: 1 (limit: 2301) 

Memory: 3.3M (peak: 3.5M) 

CPU: 284ms 

CGroup: /system.slice/nmbd.service 

└─12680 /usr/sbin/nmbd --foreground --no-process-group 

admin@debian13-server:~$ sudo apt install samba samba-common-bin -y Le paquet suivant a été installé automatiquement et n'est plus nécessaire : linux-image-6.12.73+deb13-amd64 

Veuillez utiliser « sudo apt autoremove » pour le supprimer. 

Installation de : 

samba  samba-common-bin 

je fais un nano dans /etc/samba/smb.conf pour rajouter : 

[partage] 

path = /srv/partage read only = no browsable = yes 

admin@debian13-server:~$ sudo mkdir -p /srv/partage admin@debian13-server:~$ sudo chown -R $USER:$USER /srv/partage admin@debian13-server:~$ sudo smbpasswd -a utilisateur 

New SMB password: Retype new SMB password: Failed to add entry for user utilisateur. 

admin@debian13-server:~$ sudo smbpasswd -a utilisateur 

New SMB password: 

Retype new SMB password: 

Failed to add entry for user utilisateur. 

admin@debian13-server:~$ sudo smbpasswd -a utilisateur 

New SMB password: 

Retype new SMB password: 

Failed to add entry for user utilisateur. 

>>>>>>>Mon erreur était que je n’avais pas cherché si un  compte samba était sur mon serveur et j’ai voulu ajouter un truc qui n’existe pas, alors même que je venais tout juste d’installer samba. 

Donc on corrige >>> 

admin@debian13-server:~$ sudo pdbedit -L 

>>>>>>>>>>>>> c vide, donc l’utilisateur n’était pas enregistré 

admin@debian13-server:~$ sudo adduser utilisateur 

admin@debian13-server:~$ sudo smbpasswd -a utilisateur 

admin@debian13-server:~$ 

admin@debian13-server:~$ nano /etc/samba/smb.conf 

je n’ai pas réussi a modifié mon doc car je n’ai pas fais appel a mes droits sudo 

admin@debian13-server:~$ sudo nano /etc/samba/smb.conf 

ca marche, je suis dans mon doc samba smb conf, je modifie : 

[partage] 

path = /srv/partage 

valid users = utilisateur 

read only = no 

browsable = yes 

admin@debian13-server:~$ smbclient //localhost/partage -U utilisateur 

-bash: smbclient : commande introuvable 

Je n’ai pas encore installé smbclient donc je le fais avec la ocmmande suivante 

admin@debian13-server:~$ sudo apt install smbclient -y 

Le paquet suivant a été installé automatiquement et n'est plus nécessaire : 

linux-image-6.12.73+deb13-amd64 

Veuillez utiliser « sudo apt autoremove » pour le supprimer. 

Installation de : 

Note : Je fais les tests depuis ma machine serveur car ma machine client est indispo 

Je dois naviguer dans le client samba interactif 

session setup failed: NT_STATUS_LOGON_FAILURE 

admin@debian13-server:~$ smbclient //localhost/partage -U utilisateur 

Password for [WORKGROUP\utilisateur]: 

Try "help" to get a list of possible commands. 

smb: \> ls 

.                                   D        0  Wed May  6 21:47:09 2026 ..                                  D        0  Wed May  6 21:47:09 2026 

19353424 blocks of size 1024. 16384116 blocks available 

## nfs-kernel-server 

admin@debian13-server:~$ sudo apt update 

Tous les paquets sont à jour. 

admin@debian13-server:~$ sudo apt install nfs-kernel-server -y 

admin@debian13-server:~$ sudo mkdir -p /srv/nfs_partage 

admin@debian13-server:~$ sudo chown nobody:nogroup /srv/nfs_partage 

admin@debian13-server:~$ sudo nano /etc/exports 

admin@debian13-server:~$ sudo exportfs -ra 

admin@debian13-server:~$ systemctl status nfs-kernel-server 

● nfs-server.service - NFS server and services 

Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; preset: enabled) 

Active: active (exited) since Wed 2026-05-06 22:11:56 CEST; 1min 21s ago 

Invocation: a69564cee8d84957800f4f85fbec942c 

Docs: man:rpc.nfsd(8) 

man:exportfs(8) 

Main PID: 14447 (code=exited, status=0/SUCCESS) 

Mem peak: 1.8M 

CPU: 124ms 

admin@debian13-server:~$ sudo ufw allow from 192.168.100.0/24 to any port nfs 

Rule added 

admin@debian13-server:~$ 

