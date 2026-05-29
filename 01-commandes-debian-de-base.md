# TP 1 — Découverte et prise en main de Debian 

Layla 

## Partie 2 — Prise en main du terminal 

### 2. Commandes de base 

layla@debian13-server:~$ whoami 

layla 

layla@debian13-server:~$ id 

uid=1000(layla) gid=1000(layla) 

groupes=1000(layla),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),100 (users),101(netdev),103(bluetooth) 

layla@debian13-server:~$ pwd 

/home/layla 

layla@debian13-server:~$ ls 

layla@debian13-server:~$ ls -l 

total 0 

layla@debian13-server:~$ cd /home 

layla@debian13-server:/home$ cd ~ 

layla@debian13-server:~$ 

Questions : 

- À quoi sert la commande `pwd` ? ca affiiche le chemin d'acces complet du dossier dans lequel on se trouve 

- Quelle différence entre `ls` et `ls -l` ? ls permet de lister les doc/éléments depuis le chemin on on est, tandis que ls -l liste les infos en détail du style le nombre de doc;date;permissions ect 

- Que signifie le symbole `~` le tilde sert a indiquer un dossier perso, ca indique qu'on est bien dedans 

- Quelle commande permet de revenir dans le répertoire personnel ? cd, change directory 

### 3. Manipulation de fichiers 

layla@debian13-server:~$ mkdir tp1 layla@debian13-server:~$ cd tp1 layla@debian13-server:~/tp1$ touch fichier1.txt fichier3.txt layla@debian13-server:~/tp1$ ls fichier1.txt  fichier3.txt layla@debian13-server:~/tp1$ mv fichier3.txt fichier2.txt layla@debian13-server:~/tp1$ nano fichier1.txt layla@debian13-server:~/tp1$ cat fichier1.txt 

Hello, bienvenue dans mon fichier1 format txt, ici Layla, Elle mérite un 20 la vie ne lui a pas fait de cadeaux 

MERCIIIIIIII layla@debian13-server:~/tp1$ 

## Partie 3 — Réseau 

### 1. Vérification de l’adresse IP 

layla@debian13-server:~$ ip a 

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000 

link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00 

inet 127.0.0.1/8 scope host lo 

valid_lft forever preferred_lft forever 

inet6 ::1/128 scope host noprefixroute 

valid_lft forever preferred_lft forever 

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000 

link/ether 08:00:27:65:f8:a3 brd ff:ff:ff:ff:ff:ff 

altname enx08002765f8a3 

inet 192.168.1.49/24 brd 192.168.1.255 scope global dynamic noprefixroute enp0s3 valid_lft 83449sec preferred_lft 72649sec 

inet6 2a02:8428:633a:ff01:a00:27ff:fe65:f8a3/64 scope global dynamic mngtmpaddr proto kernel_ra 

valid_lft 604552sec preferred_lft 604552sec 

inet6 2a02:8428:633a:ff01:b597:b89f:f0fe:20e2/64 scope global dynamic mngtmpaddr noprefixroute 

valid_lft 604552sec preferred_lft 604552sec 

inet6 fe80::d07c:fd69:77c5:fcb5/64 scope link 

valid_lft forever preferred_lft forever 

layla@debian13-server:~$ ip route 

default via 192.168.1.1 dev enp0s3 proto dhcp src 192.168.1.49 metric 1002 192.168.1.0/24 dev enp0s3 proto dhcp scope link src 192.168.1.49 metric 1002 layla@debian13-server:~$ 

### 2. Test de connectivité 

## yacin@DESKTOP-358ABA8 MINGW64 ~ 

$ ping 192.168.1.49 

Envoi d’une requête 'Ping'  192.168.1.49 avec 32 octets de données : Réponse de 192.168.1.49 : octets=32 temps=2 ms TTL=64 Réponse de 192.168.1.49 : octets=32 temps=1 ms TTL=64 Réponse de 192.168.1.49 : octets=32 temps=2 ms TTL=64 Réponse de 192.168.1.49 : octets=32 temps=1 ms TTL=64 

Statistiques Ping pour 192.168.1.49: 

Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%), 

Durée approximative des boucles en millisecondes : 

Minimum = 1ms, Maximum = 2ms, Moyenne = 1ms 

Questions : 

- Que signifie une réponse au ping ? ca veut dire que la machine est joignable, on recoit bien ses reponses aux requetes icpm, 

- Quelle différence entre ping 8.8.8.8 et ping google.com ? ping 8.8.8.8 c pour tester la connexion vers une adresse ip. ping google.com passe par me DNS, domain name system pr recevoir les paquets. 

- Que faire si le ping vers une adresse IP fonctionne mais pas vers un nom de domaine ? Il y a un soucis au niveau du DNS. Il faudrat alors ping sous la forme ip et verifier la configuration du DNS 

- À quoi correspond la passerelle par défaut ? la passerelle est l adresse par defaut dans lequel l utilisateur envoit des paquets quand destination non locale. 

- Quel est son rôle dans le réseau ? c est pour passer de local a hors local comme en réseau. 

## Partie 4 — Installation d’un serveur web 

### 1. Mise à jour du système 

layla@debian13-server:~$ sudo apt update 

[sudo] Mot de passe de layla : 

Atteint : 1 http://deb.debian.org/debian trixie InRelease 

Réception de : 2 http://deb.debian.org/debian trixie-updates InRelease [47,3 kB] 

- Réception de : 3 http://security.debian.org/debian security trixie-security InRelease [43,4 kB] 

- Réception de : 4 http://security.debian.org/debian security trixie-security/main Sources [127 kB] 

- Réception de : 5 http://security.debian.org/debian security trixie-security/main amd64 Packages [123 kB] 

340 ko réceptionnés en 0s (867 ko/s) 

Tous les paquets sont à jour. 

layla@debian13-server:~$ sudo apt upgrade 

Sommaire : 

Mise à niveau de : 0. Installation de : 0Supprimé : 0. Non mis à jour : 0 

### 2. Installation d’Apache 

layla@debian13-server:~$ sudo apt install apache2 -y 

Installation de : 

apache2 

## Installation de dépendances : 

apache2-bin              libaprutil1-ldap  liblua5.4-0       libsasl2-modules-db apache2-data             libaprutil1t64    libnghttp3-9      libssh2-1t64 apache2-utils            libcurl4t64       librtmp1          ssl-cert libapr1t64               libldap-common    libsasl2-2 

libaprutil1-dbd-sqlite3  libldap2          libsasl2-modules 

Paquets suggérés : 

apache2-doc              www-browser                        libsasl2-modules-otp 

apache2-suexec-pristine  libsasl2-modules-gssapi-mit        libsasl2-modules-sql 

| apache2-suexec-custom  | libsasl2-modules-gssapi-heimdal 

ufw                      libsasl2-modules-ldap 

Sommaire : 

Mise à niveau de : 0. Installation de : 19Supprimé : 0. Non mis à jour : 0 

Taille du téléchargement : 3 529 kB 

Espace nécessaire : 11,6 MB / 17,4 GB disponible 

Réception de : 1 http://deb.debian.org/debian trixie/main amd64 libapr1t64 amd64 1.7.5-1 [104 kB] 

.......... 

.......... 

.......... 

.......... 

.......... 

.......... 

Created symlink '/etc/systemd/system/multi-user.target.wants/apache2.service' → '/usr/lib/systemd/system/apache2.service'. 

Created symlink '/etc/systemd/system/multi-user.target.wants/apachehtcacheclean.service' → '/usr/lib/systemd/system/apache-htcacheclean.service'. 

Traitement des actions différées (« triggers ») pour man-db (2.13.1-1) ... Traitement des actions différées (« triggers ») pour libc-bin (2.41-12+deb13u2) ... 

