
WINDOWS SERVER 2016

------------------------------------------------------------------

# Mettre IP statique
- Aller dans la petite planète en bas à droite
- Paramètres réseaux > Centre réseau et partage > Ethernet 0 (carte réseau) > Propriétés > Protocole Internet version 4 (TCP/IPv4)
- Exemple IP : 192.168.1.10 - Masque : 255.255.255.0 - Passerelle : 192.168.1.254 - DNS préféré : 127.0.0.1 - DNS auxiliaire : 192.168.1.10


- Gestionnaire de serveur > Gérer > Ajouter des rôles et fonctionnalités

# Installation AD DS

- Ajouter une nouvelle forêt qui sera notre domaine
- Redémarrer
- Dans "CE PC", clique droit, propriétés, modifier.
- Redémarrer et se connecter sur DOMAINE\Administrateur

# Installation DNS
- L'installer et aller dans Outils > DNS
- Cliquer sur notre domaine et cliquer sur Zones de recherche inversés > Nouvelle zone
- Tout par défaut
- Exemple d'ID réseau : 192.168.1. et ne rien mettre dans le quatrième octet
- Terminer
- Clique droit sur notre zone et nouveau pointeur (PTR)
- Parcourir jusqu'à notre domaine hôte (A), nom de l'hôte doit ressembler à : nommachine.domaine.local
- Redémarrer
- Vérifier dans CMD avec nslookup

# Installation DHCP

- Aller dans Terminer configuration DHCP
- Tout par défaut
- Aller dans Outils > DHCP > IPv4 > Nouvelle étendue
- Exemple de Nom : 192.168.1.0 - Adresse IP de début : 192.168.1.11 - Adresse IP de fin : 192.168.1.50
- Tout par défaut
- Pour Routeur (passerelle par défaut) on peut ajouter la passerelle comme par exemple : 192.168.1.254 qu'on a renseigné plus haut
- Terminer


Si PC Server :

# Si DHCP pas installé dans le premier PC, mettre une adresse statique

# Installer DNS + DHCP sur PC 1

- Renommer son PC et ajouter le domaine
- Dans "CE PC", clique droit, propriétés, modifier.
- Redémarrer et se connecter sur DOMAINE\Administrateur

- (Re)mettre IP automatique
- Vérifier dans CMD avec ipconfig
