

C’est quoi l'infrastructure qu'on a mise en place ? 

Le serveur Debian, c’est le chef du réseau. Il gère tout : il distribue les IPs, répond aux noms de domaine, synchronise l'heure, et filtre le trafic. Le truc c’est qu’il lui manque des services pour fonctionner correctement en assurant la protection, l’acces aus ressources, la cohérence, la connexion, la synchronisation de l’heure ect. 
La logique globale — chaque service dépend des autres. DNS doit fonctionner pour Kerberos (Samba). NTP doit fonctionner pour que les logs soient cohérents. Le firewall doit laisser passer les bons ports pour que tout communique. C'est une chaîne, pas des services indépendants. 
TCP vs UDP en résumé — UDP c'est rapide mais sans garantie (DHCP, DNS, NTP), TCP c'est fiable avec vérification de livraison (SSH, FTP, Samba). C'est pourquoi les services critiques utilisent TCP et les services de découverte/temps utilisent UDP. 
FTP vs SFTP — c'est volontairement dans le TP pour montrer le contraste : FTP envoie tout en clair (mot de passe visible sur le réseau !), SFTP chiffre tout. En production, FTP est abandonné 
au profit de SFTP. J'ai déployé un DHCP qui distribue automatiquement les configurations réseau aux clients via le 
protocole DORA Chrony, pour synchroniser le temps ... 
 
Résumé :  
J'ai mis en place un serveur Debian avec une IP statique 192.168.100.107. Sur ce serveur j'ai installé les services réseau de base : DHCP pour distribuer les adresses IP automatiquement, DNS avec bind9 pour la résolution de noms avec une zone locale mondomaine.local, et NTP avec chrony pour synchroniser l'heure. J'ai ensuite sécurisé l'accès avec UFW en appliquant une politique deny par défaut et en n'ouvrant que les ports nécessaires. 
 
Site utilisé : 
https://debian-facile.org/ 
 
A chaque pré installation, on va faire des apt upgrate et apt upgrade : 
apt update # Rafraîchit la liste des logiciels disponibles 
apt upgrade # Met à jour tous les logiciels déjà installés 
puis les apt install (de bind9, dhcp...) 
 
systemctl c'est la commande qui gère tous les services qui tournent sur ton serveur : 
start – stop – restart -enable -reload – disable 
ex : systemctl start bind9 
 
Organisation 
1. apt install <paquet>        → installer 
2. nano /etc/...               → configurer 
3. systemctl restart <service> → appliquer 
4. systemctl enable <service>  → activer au démarrage 
5. systemctl status <service>  → vérifier 