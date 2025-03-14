# Configuration Réseau

## 1. Câble Console et Sécurisation

Le câble console permet de connecter un ordinateur à un switch pour effectuer la première configuration.

- **Commande `enable`** : Permet de passer en mode privilégié. Ce mode permet de consulter diverses configurations via la commande `show`.

- **Commande `configure terminal`** : Permet de passer en mode de configuration du switch.

### Sécurisation du Switch
1. **Désactivation de la recherche de nom de domaine** :  
   Pour éviter toute attente en cas de mauvaise manipulation, on désactive la recherche de noms avec la commande :  
   `no ip domain-lookup`.

2. **Sécurisation du mode privilégié** :  
   Définir un mot de passe pour le mode privilégié :  
   `enable secret <mot_de_passe>`.  
   Puis crypter ce mot de passe :  
   `service password-encryption`.

3. **Sécurisation de l'accès via le câble console** :  
   Aller dans la configuration du port console :  
   `line console 0`.  
   Définir un mot de passe :  
   `password <mot_de_passe>`.  
   Et activer la vérification du mot de passe :  
   `login`.

## 2. Accès à Distance

Pour accéder à distance au switch via SSH, il faut configurer le protocole SSH.

### Configuration de SSH :
1. **Nom du Switch** :  
   `hostname <nom_du_switch>`.

2. **Création d'un utilisateur** :  
   `username <nom_utilisateur> password <mot_de_passe>`.

3. **Configuration de la ligne VTY** :  
   Pour permettre deux connexions simultanées via SSH :  
   `line vty 0 1`.  
   Configurer le protocole SSH sur cette ligne :  
   `transport input ssh`.  
   Puis indiquer l'utilisation d'un mot de passe local :  
   `login local`.

4. **Nom de domaine** :  
   Création du nom de domaine nécessaire à la génération de la clé SSH :  
   `ip domain-name <nom_du_domaine>`.

5. **Génération de la clé SSH** :  
   La taille de la clé RSA peut être choisie, généralement 2048 bits :  
   `crypto key generate rsa general-keys modulus 2048`.

## 3. Sécurisation des Ports

La sécurisation des ports permet de restreindre l'accès aux ports du switch à des adresses MAC spécifiques.

### Configuration des ports :
1. **Mode access** :  
   Configurer le port pour qu'il soit en mode access, ce qui signifie qu'il est connecté à un périphérique final (PC, imprimante, etc.) :  
   `switchport mode access`.

2. **Activation de la sécurité des ports** :  
   `switchport port-security`.

3. **Assignation d'une adresse MAC** :
   - **Manuelle** :  
     `switchport port-security mac-address <adresse_mac>`.
   - **Automatique** (sticky) :  
     `switchport port-security mac-address sticky`.

4. **Définir le nombre maximum d'adresses MAC autorisées sur un port** :  
   `switchport port-security maximum <nombre>`.

5. **Comportement en cas de violation** :  
   Lorsqu'une adresse MAC non autorisée est détectée, il y a plusieurs options :  
   - **Shutdown** : Désactive l'interface et nécessite une intervention manuelle pour la réactiver.  
     `shutdown` puis `no shutdown` pour réactiver.
   - **Protect** : Bloque les trames avec des adresses MAC non autorisées.  
   - **Restrict** : Bloque les trames non autorisées et envoie une alerte SNMP.  
     `switchport port-security violation <shutdown|protect|restrict>`.

6. **Consultation de la sécurité des ports** :  
   Pour afficher la liste des interfaces sécurisées, le nombre de violations, etc. :  
   `show port-security`.

## 4. Configuration des VLANs

Les VLANs permettent de créer plusieurs sous-réseaux sur un même câble physique.

### Types de VLAN :
- **VLAN par port** : Le switch associe le port à un VLAN indépendamment de l'adresse MAC.
- **VLAN par adresse MAC** : Le switch associe l'adresse MAC du périphérique à un VLAN.

### Configuration d'un VLAN :
1. **Création d'un VLAN** :  
   `vlan <numéro_du_vlan>`.

2. **Affectation d'un VLAN à un port** :  
   - Configurer le port en mode access pour un périphérique final :  
     `switchport mode access`.  
   - Associer un VLAN au port :  
     `switchport access vlan <numéro_du_vlan>`.

3. **Mode trunk pour l'interconnexion entre switches** :  
   Pour autoriser plusieurs VLANs à circuler sur une même interface :  
   `switchport mode trunk`.

4. **Nombre maximal de VLANs sur un trunk** :  
   Le nombre maximum de VLANs qu'un port trunk peut supporter est 4095.

## 5. Routage Inter-VLAN

Le routage inter-VLAN permet de relier plusieurs VLANs, nécessitant un routeur ou un switch de niveau 3.

### Configuration du "Routeur-on-Stick" :
1. **Interface virtuelle pour chaque VLAN** :  
   `interface gigabitethernet0/0.<numéro_du_vlan>`.  
   Attribuer une adresse IP à cette interface virtuelle :  
   `ip address <adresse_ip> <masque>`.

2. **Encapsulation des trames** :  
   Pour spécifier que les trames sont de type VLAN 802.1Q :  
   `encapsulation dot1Q <numéro_du_vlan>`.

## 6. Routage Dynamique

Le routage dynamique permet de gérer la transmission des informations de routage entre les routeurs. Il existe plusieurs protocoles, dont RIP, OSPF et EIGRP.

### 6.1. Routage avec RIP (Routing Information Protocol)

RIP utilise un algorithme de vecteur de distance et fonctionne avec des adresses IP classful.

- **Configuration RIP** :  
  `router rip`.  
  `version 2` (pour la version 2, plus sécurisée).  
  `network <adresse_ip_du_réseau>` (définir les réseaux directement connectés).

- **Configuration d'une route par défaut** :  
  `default-information originate`.

### 6.2. Routage avec OSPF (Open Shortest Path First)

OSPF utilise un algorithme de plus court chemin (Dijkstra) pour déterminer la route optimale. Il prend en compte la bande passante des liens et fonctionne par zones.

- **Configuration OSPF** :  
  `router ospf 1`.  
  `network <adresse_du_réseau> <masque_inversé> area <numéro_de_zone>`.

- **Configuration d'une route par défaut** :  
  `default-information originate`.

### 6.3. Routage avec EIGRP (Enhanced Interior Gateway Routing Protocol)

EIGRP est un protocole de routage propriétaire Cisco qui utilise un calcul plus précis que RIP et OSPF.

- **Configuration EIGRP** :  
  `router eigrp 1`.  
  `network <adresse_du_réseau>`.

## 7. Access Control List (ACL)

Les ACL permettent de filtrer le trafic réseau en fonction de l'adresse source, de l'adresse de destination et du protocole.

### 7.1. ACL Standard

Une ACL standard filtre uniquement sur l'adresse source.

- **Création d'une ACL standard** :  
  `access-list <numéro (1-99)> deny/permit <adresse_source>`.  
  `ip access-list standard <nom> deny/permit <adresse_source>`.

- **Application de l'ACL à une interface** :  
  `ip access-group <numéro/nom> <in|out>`.

### 7.2. ACL Étendue

Les ACL étendues permettent un filtrage plus précis (source, destination, protocole, ports).

- **Création d'une ACL étendue** :  
  `access-list <numéro (100-199)> deny/permit <protocole> <adresse_source> <masque_inversé> <adresse_destination> <masque_inversé>`.  
  Par exemple :  
  `access-list 100 deny tcp any host 192.168.1.1 eq 80`.

## 8. Network Address Translation (NAT)

NAT permet de traduire les adresses IP privées en publiques et vice versa.

### 8.1. NAT Statique

Le NAT statique permet d’associer une adresse IP privée à une adresse IP publique.

- **Configuration NAT statique** :  
  `ip nat inside source static <ip_privée> <ip_publique>`.  
  Sur l'interface entrant : `ip nat inside`.  
  Sur l'interface sortante : `ip nat outside`.

### 8.2. NAT Dynamique

Le NAT dynamique permet d'associer plusieurs adresses privées à une ou plusieurs adresses publiques.

- **Configuration NAT dynamique** :  
  `access-list <numéro> permit <adresse_ip_source> <masque>`.  
  `ip nat pool <nom_pool> <ip_début> <ip_fin> netmask <masque>`.  
  `ip nat inside source list <nom_list> pool <nom_pool>`.  
  Ajouter `overload` pour la surcharge.

### 8.3. Port Address Translation (PAT)

Le PAT permet d'utiliser des numéros de port pour traduire plusieurs adresses privées en une seule adresse publique.

- **Configuration PAT** :  
  `ip nat inside source list <numéro_list> interface <interface> overload`.
