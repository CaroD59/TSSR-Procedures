# Configuration réseau

## Câble console et sécurisation

Le câble console permet de connecter un ordinateur au switch pour effectuer la première configuration.  
> La commande **enable** permet d'entrer en mode privilégié. 

Le mode privilégié permet de consulter les différentes configurations à l'aide de la commande **show**, suivie de ce que l'on souhaite consulter.  
> **configure terminal** permet de passer en mode de configuration du switch.

La première action à réaliser en mode configuration est de désactiver la recherche de nom de domaine, afin d'éviter des délais inutiles en cas de fausse manipulation :  
**no ip domain-lookup**.

Ensuite, il est nécessaire de sécuriser le switch en attribuant un mot de passe pour entrer en mode privilégié :  
**enable secret _mot_de_passe_**. Une fois le mot de passe défini, il faut le crypter :  
**service password-encryption**.

Il faut également sécuriser l'accès au switch via le câble console. Pour cela, on se place en configuration de ce câble :  
**line console 0**. Puis, on définit un mot de passe :  
**password _mot_de_passe_**.  
Ensuite, il faut indiquer **login** pour valider la vérification par mot de passe.

## Accès à distance (SSH)

Pour pouvoir administrer le switch à distance, il faut configurer le protocole SSH. Voici les étapes :

1. **Nommer le switch** :  
   En mode configuration :  
   **hostname _nom_du_switch_**

2. Créer un nom d'utilisateur et un mot de passe :  
   **username _username_ password _mot_de_passe_**

3. Accéder à la ligne de connexion :  
   **line vty 0 1** (Les numéros 0 et 1 correspondent au nombre de connexions simultanées autorisées, ici 2.)

4. Configurer le transport pour SSH :  
   **transport input ssh** (Cela indique au switch qu'on utilisera SSH sur cette ligne.)

5. Définir la méthode de connexion :  
   **login local** (Cela indique au switch de vérifier les informations d'identification dans sa table d'utilisateurs.)

6. Créer un nom de domaine pour la clé de chiffrement :  
   **ip domain-name _nom_du_domaine_**

7. Générer la clé SSH :  
   **crypto key generate rsa general-keys modulus _taille_**  
   (La taille indique le nombre de bits souhaité pour la clé.)

## Sécurisation des ports

La sécurisation des ports d'un switch consiste à autoriser certaines adresses MAC (ou plusieurs) et à bloquer les autres. Voici comment procéder :

1. Mettre l'interface en mode **access** :  
   **switchport mode access**  
   (Cela précise que l'interface est connectée à un dispositif final.)

2. Activer le service de **port security** :  
   **switchport port-security**

3. Assignation d'une adresse MAC à l'interface :  
   - **Manuelle** :  
     **switchport port-security mac-address _adresse_mac_**  
   - **Automatique (sticky)** :  
     **switchport port-security mac-address sticky**  
     (Le switch attribuera l'adresse MAC de la première trame qu'il recevra à ce port.)

4. Définir le nombre maximal d'adresses MAC autorisées sur ce port :  
   **switchport port-security maximum _nombre_**

5. Choisir la réaction du switch en cas de violation de sécurité :  
   - **shutdown** :  
     L'interface est désactivée et doit être réactivée manuellement avec **no shutdown**.
   - **protect** :  
     Les trames avec des adresses MAC non autorisées sont bloquées.
   - **restrict** :  
     Les violations sont bloquées et une alerte SNMP est envoyée. Le compteur de violations est incrémenté.

6. Appliquer la méthode de violation :  
   **switchport port-security violation _méthode_**

7. Pour consulter l'état de la sécurité des ports :  
   **show port-security**

## Configuration des VLANs

Les VLANs sont utilisés lorsqu'il y a plusieurs sous-réseaux sur un même câble physique.  
Il existe deux types de VLANs :

- **Par port** : Le switch associe un port à un VLAN sans tenir compte de l'appareil connecté.
- **Par adresse MAC** : Le switch assigne un VLAN en fonction de l'adresse MAC de la machine connectée.

### Création d'un VLAN

Pour créer un VLAN :  
**vlan _numéro_du_vlan_**

### Affectation d'un VLAN à un port

1. **Mode Access** :  
   **switchport mode access**  
   (Cela précise que le port est connecté à un dispositif final.)

2. **Attribution du VLAN** :  
   **switchport access vlan _numéro_**

### Configuration d'un Trunk

Les interfaces reliant les équipements réseau doivent être en mode trunk pour autoriser les trames de plusieurs VLANs à passer :  
**switchport mode trunk**

**Limite** : Un maximum de 255 VLANs peut être configuré en simultané.

## Routage Inter-VLAN

Pour connecter plusieurs VLANs, un routeur est nécessaire. Le montage le plus courant est un **"routeur-on-stick"** : un seul câble relie le routeur au switch qui regroupe les différents VLANs.

### Création d'interfaces virtuelles

Pour chaque VLAN, il faut créer une interface virtuelle sur le routeur :  
**interface gigabitEthernet0/0._numéro_du_vlan_**

1. Ne pas attribuer d'adresse IP à l'interface physique mais l'activer :  
   **no shutdown**

2. Configurer l'adresse IP de l'interface virtuelle :  
   **ip address _adresse_ip_ _masque_**

3. Encapsuler les trames VLAN avec la commande :  
   **encapsulation dot1Q _numéro_du_vlan_**

## Routage dynamique (RIP)

Le **RIP** permet aux routeurs de communiquer entre eux pour déterminer les meilleures routes. Il utilise un algorithme de vecteur de distance basé sur le nombre de "sauts" (stations intermédiaires). 
Il fonctionne sur la base des adresses IP classful, en regardant la configuration des réseaux: **show running-config** ou **show ip rip database** on remarquera que les adresses réseaux que nous lui avons donné ne respecte pas forcément le sous-découpage que nous avons réalisé. Cependant en regardant les **routes** grâce à la commande: **show ip route** les adresses réseau ainsi que leur masque correspondant sont indiqué correctement.

### Configuration du RIP

1. Entrer en mode configuration RIP :  
   **router rip**

2. Sélectionner la version 2 (plus sécurisée et supporte le VLSM) :  
   **version 2**

3. Ajouter les réseaux auxquels le routeur a accès :  
   **network _adresse_ip_du_réseau_**

4. Si une route par défaut doit être transmise :  
   **default-information originate**

## Routage dynamique (OSPF)

**OSPF** fonctionne selon un algorithme de plus court chemin (Dijkstra), qui prend en compte les coûts (poids) des chemins. Il est également basé sur un système de zones pour limiter la communication dans un réseau large. Il prend en compte la vitesse d'un câble et l'utilise comme **poids**; additionnant tous les poids jusqu'à destination. Le chemin qui a le poids le plus faible est sélectionné pour être la route qui sera mise dans la table de routage.

### Configuration de OSPF

1. Entrer en mode configuration OSPF :  
   **router ospf 1**

2. Ajouter les réseaux et leurs zones :  
   **network _adresse_du_réseau_ _masque_inversé_ area _numéro_de_zone_**

3. Si une route par défaut doit être transmise :  
   **default-information originate**

## Routage dynamique (EIGRP)

**EIGRP** est un protocole propriétaire de Cisco qui prend en compte la bande passante et le délai des interfaces pour déterminer la meilleure route. Il est impossible de le déployer si un appareil de la topologie n'est pas de Cisco.

### Configuration de EIGRP

1. Entrer en mode configuration EIGRP :  
   **router eigrp 1**

2. Ajouter les réseaux auxquels le routeur a accès :  
   **network _ip_du_réseau_**

3. La valeur **1** dans la commande d'EIGRP doit être la même sur tous les routeurs de la topologie pour qu'ils puissent communiquer.

## Routage inter-protocoles

Pour faire communiquer deux protocoles de routage différents, il faut utiliser la commande :  
**redistribute _nom_du_protocole_**

## Access Control List (ACL)

Les **ACL** permettent de restreindre l'accès à certaines parties du réseau, selon l'adresse IP source ou d'autres critères.

### Types d'ACL

- **Standard** : Restrictions basées uniquement sur l'adresse IP source.
- **Extended** : Restrictions plus détaillées, y compris l'adresse source et de destination, le protocole, et le port.

### Configuration d'ACL standard

1. Créer une ACL standard :  
   **access-list _numéro_ deny/permit _adresse_source_**

2. Appliquer l'ACL à une interface :  
   **ip access-group _numéro/nom_ in/out**

### Configuration d'ACL étendue

1. Créer une ACL étendue :  
   **access-list _numéro_ deny/permit _protocole_ _adresse_source_ _masque_inversé_ _adresse_destination_ _masque_inversé_ _port_**

2. Appliquer l'ACL :  
   **ip access-group _numéro_ in/out**

### Remarques

- Les ACL sont traitées de la règle la plus précise à la plus générale.
- Il est recommandé de conserver une copie des ACL dans un bloc-notes pour pouvoir les réutiliser ou les modifier facilement.

eq = equal
neq = non equal
lt = less than
gt = greater than

## Network Address Translation (NAT)

NAT permet de traduire les adresses IP privées en adresses IP publiques pour l'accès à Internet. Il existe trois types de NAT :

1. **NAT statique** : Une adresse privée est mappée à une adresse publique.
2. **NAT dynamique** : Plusieurs adresses privées sont mappées à une ou plusieurs adresses publiques.
3. **PAT (Port Address Translation)** : Plusieurs adresses privées sont mappées à une seule adresse publique, avec un port unique pour chaque connexion.

### Configuration de NAT statique
Il permet d'attribuer une adresse privée à une adresse publique. 

1. **ip nat inside source static _adresse_privée_ _adresse_publique_**

Sur l'interface d'entrée dans le routeur on précise **ip nat inside** pour indiquer au routeur que l'adresse à traduire arrivera de ce port.
Sur l'interface de sortie (vers le réseau WAN) on indique **ip nat outside** pour dire au routeur sur quel port envoyé l'adresse traduite.

### Configuration de NAT dynamique
Le NAT dynamique permet d'associer plusieurs adresses privées à une ou plusieurs adresses publiques. Cependant si toutes les adresses IP publiques sont occupées, la machine suivante qui demande accès à internet ne pourra pas obtenir d'adresse tant qu'un autre appareil ne se sera pas déconnecté.

1. Créer une access-list pour les réseaux autorisés à sortir :  
   **access-list _numéro_ permit _ip_source_ _masque_**

2. Créer un pool d'adresses publiques :  
   **ip nat pool _nom_du_pool_ _ip_de_début_ _ip_de_fin_ netmask _masque_**

3. Appliquer la traduction :  
   **ip nat inside source list _nom_de_l'access-list_ pool _nom_du_pool_**

Dans le cas où nous avons plus d'adresses privées ayant besoin d'accéder à internet (on parle de nat avec surcharge), on ajoute **overload** à la fin de la dernière commande.

### Configuration de PAT
Le PAT permet de déplacer le problème en attribuant à chaque demande de traduction un numéro de port aléatoire et unique qui permettra d'identifier quel est la machine qui souhaite communiquer.

1. Créer une access-list pour les réseaux autorisés à sortir.
2. Appliquer la commande PAT :  
   **ip nat inside source list _nom_de_l'access-list_ interface _nom_de_l'interface_ overload**
