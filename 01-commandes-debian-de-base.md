# Les commandes Debian

## 1. Objectif du TP
Ce TP a pour objectif de prendre en main un système Linux Debian afin de comprendre les commandes de base, la gestion des fichiers, les notions fondamentales de réseau ainsi que l’installation d’un service serveur (Apache).

L’objectif est de développer les compétences essentielles d’un administrateur système : utilisation du terminal, diagnostic réseau et installation de services.

## 2. Prise en main du terminal
### 2.1 Commandes de base
Dans un environnement Linux, le terminal permet d’interagir directement avec le système via des commandes.

**Les commandes suivantes ont été utilisées :**
whoami : affiche l’utilisateur connecté
id : affiche les identifiants utilisateur (UID, GID, groupes)
pwd : affiche le chemin du répertoire courant
ls : liste les fichiers et dossiers
ls -l : affiche les fichiers avec détails (permissions, taille, date, propriétaire)
cd : permet de changer de répertoire

Le symbole ~ représente le répertoire personnel de l’utilisateur.

### 2.2 Analyse des commandes

Commande pwd
Elle permet d’afficher le chemin complet du répertoire dans lequel l’utilisateur se trouve.

Différence entre ls et ls -l
ls affiche simplement les éléments présents dans un dossier, tandis que ls -l fournit une vue détaillée incluant les permissions, la taille et la date de modification.

Signification de ~
Le symbole ~ correspond au répertoire personnel de l’utilisateur connecté.

Retour au répertoire personnel
La commande cd ou cd ~ permet de revenir dans le dossier personnel.

### 2.3 Manipulation de fichiers

Cette étape permet de comprendre la gestion des fichiers et dossiers sous Linux.

**Commandes utilisées :**
mkdir tp1
cd tp1
touch fichier1.txt fichier3.txt
ls
mv fichier3.txt fichier2.txt
nano fichier1.txt
cat fichier1.txt

**Explication :**
mkdir : création d’un dossier
touch : création de fichiers vides
mv : renommage ou déplacement de fichier
nano : édition de fichier en mode terminal
cat : affichage du contenu d’un fichier


Cette manipulation permet de comprendre les opérations de base sur les fichiers sous Linux.

## 3. Partie réseau
### 3.1 Vérification de la configuration réseau

La commande ip a permet d’afficher la configuration réseau de la machine (interfaces, adresses IP, état des connexions).

La commande ip route permet d’afficher la table de routage, notamment la passerelle par défaut utilisée pour accéder aux réseaux externes.

### 3.2 Test de connectivité

Le test de connectivité est réalisé avec la commande ping.
Cette commande envoie des paquets ICMP afin de vérifier la disponibilité d’une machine sur le réseau.

Analyse des résultats possibles :

**Signification d’un ping réussi**
Un ping réussi signifie que la machine cible est joignable et répond aux requêtes réseau.

**Différence entre ping 8.8.8.8 et ping google.com**
- ping 8.8.8.8 teste directement une adresse IP
- ping google.com nécessite une résolution DNS avant d’atteindre la machine

**Problème si IP fonctionne mais pas le nom de domaine**
Cela indique un dysfonctionnement du service DNS.

**Rôle de la passerelle par défaut**
La passerelle permet d’acheminer les paquets vers des réseaux externes lorsque la destination n’est pas locale.

## 4. Installation d’un serveur web
### 4.1 Mise à jour du système

Avant toute installation, le système est mis à jour afin de garantir la sécurité et la compatibilité des paquets.

```bash
sudo apt update
sudo apt upgrade

Cette étape permet de synchroniser la liste des paquets et d’installer les dernières mises à jour disponibles.

### 4.2 Installation du serveur Apache

Le serveur web Apache2 a été installé à l’aide de la commande suivante :
```bash
sudo apt install apache2 -y'''

Présentation
Apache est un serveur web permettant d’héberger des sites accessibles via un navigateur. Il fonctionne en arrière-plan sous forme de service système.

Résultat
Lors de l’installation, plusieurs dépendances ont été installées automatiquement. Le service Apache a été activé et configuré pour démarrer automatiquement avec le système.

## 5. Conclusion

Ce TP a permis de prendre en main un environnement Debian et de maîtriser les bases essentielles de l’administration système.

Les compétences développées concernent :

l’utilisation du terminal Linux
la gestion des fichiers et dossiers
les diagnostics réseau
l’installation et la configuration d’un service serveur

Ces notions constituent les bases indispensables pour l’administration d’un système Linux en environnement professionnel.