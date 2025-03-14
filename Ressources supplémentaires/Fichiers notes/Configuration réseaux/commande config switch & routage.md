# Configuration réseau

## Câble console et sécurisation
- Le câble console permet de connecter un ordinateur au switch pour réaliser la première configuration.
> la commande **enable** permet de rentré en mode privilégié

Le mode privilège permet de voir les différentes configuration à l'aide de la commande **show** suivie de ce que l'on souhaite consulté.

>**configure terminal** permet de rentré en mode de configuration du switch

la première chose à faire en mode configuration est de désactivé la recherche de nom de domaine pour éviter de perdre du temps lors de fausse manipulation: **no ip domain-lookup**

il faut ensuite sécurisé le switch, pour cela nous devont donner un mot de passe pour entré en mode privillègié:
**enable secret _mot de passe_**. une fois le mot de passe crée il faut le crypté: **service password-encryption**

 une fois fait nous devons également sécurisé l'accès au switch sur le câble console, pour cela allons dans la configuration de ce câble: **line console 0** et y définir un mot de passe: **password _mot de passe_**
 
indiquer ensuite **login** pour valider la vérification par mot de passe

## Accès à distance 

pour pouvoir intervenir sur le switch à distance nous devons configuré le protocole SSH:

il faut d'abord donner un nom à notre switch: *en mode config* **hostname _nom du switch_**

puis crée un nom d'utilisateur et son mot de passe:
**username _username_ password _password_**

ensuite accéder à la ligne de connexion: **line vty 0 1** le 0 et le 1 correspondent au nombre de connexion en simultané que le switch autorisera ici 2 car on compte à partir de 0)

**transport input ssh** indique au switch que nous utiliseront le protocole ssh sur cette ligne

taper **login local** pour indiquer au switch de cherche dans ça table d'utilisateur le mot de passe correspondant.

crée ensuite un nom de domaine qui servira à crée la clef de chiffrement: **ip domain-name _nom du domaine_**

il ne nous reste plus qu'a crée notre clef SSH: 
**crypto key generate rsa general-keys modulus _size_**

size indique la taille de la clef en bits que nous voulons.

## Sécurisation des ports

la sécurisation des ports d'un switch consiste à autorisé une adresse mac (ou plusieurs) et à bloquer les autres.
il faut d'abord que l'interface soit en mode access (précisant que cet interface est connecté à un end device) **switchport mode access**

ensuite activé le service de port-sécurity: **switchport port-sécurity**

nous avons ensuite deux méthodes pour assigner une adresse MAC à cette interface:

- manuel: **switchport port-security mac-address _adresse mac_**
- automatique: **switchport port-security mac-address sticky**
Le switch attribuera l'adresse mac de la première trame qu'il recevra à ce port.

pour autorisé plusieurs adresse mac sur un même port; il faut indique le nombre maximum d'adresse que nous voulons: **switchport port-security maximum _nombre_**

Nous pouvons ensuite définir comment l'interface doit réagir si une adresse MAC non autorisé tente de communiqué sur le réseau:

- **shutdown**
	- elle désactivera l'interface et il faudra l'intervention d'un technicien pour la réactiver: **shutdown** pour l'éteindre manuellement puis **no shutdown** pour la réactivé.

- **protect**
	- toute les trames ayant des adresse MAC non autorisé seront bloqué et celles autorisé passeront.

- **restrict**
	- Alerte SNMP envoyé et le compteur de violation est incrémenté.
	- cette méthode permet de faire remonter l'information d'une violation au serveur de supervision (requete SNMP) et d'indiquer le nombre de violation qui ont eu lieu.

pour ajouter ces réglage: **swichport port-security violation _nom de la méthode_**

pour consulté la liste des interfaces protéger, le nombre de violation, le nombre maximum d'adresse mac autorisé et le nom d'adresse actuellement autorisé ainsi que la méthode de violation en place: **show port-security**

## Configuration des VLAN

les VLAN sont utilisé quand nous avons **plusieurs sous-réseaux mais un seul câble physique.** 

il existe deux types de vlan:
- **par port:**
	- le switch associera le port à un vlan sans tenir compte de la machine qui y es connecté

- **par adresse MAC** 
	- peut importe le port le switch regardera l'adresse MAC de la machine connecté et l'affectera au vlan qui lui correspond.

**Les vlan par adresse mac sont assez peu utilisé bien que plus souple**

pour crée un vlan: **vlan _numero du vlan_**

pour qu'un vlan fonctionne et qu'une machine ne puisse accéder que au vlan qui lui attribué il faut indiquer au port qu'il est connecté à un **end device** et non à un autre équipement réseau:

**switchport mode access** => indique au switch une connexion à un end device
**switchport access vlan _numero_** => indique le vlan attribué à ce port

pour les interface reliant les appareils réseaux il doivent être en mode trunk pour autorisé le passage des trames venant de **plusieurs vlans:**

**switchport mode trunk**

le nombre MAXIMUM de vlan en simultané est **250.**

## Routage inter-vlan 

pour connecter plusieurs vlan, il faut configurer un routeur.
Le montage le plus courant es un **"routeur-on-stick"** c'est à dire qu'un seul câble relis le routeur au switch qui regroupe les différents vlan.

pour cela nous devons crée des interfaces virtuelles:
**interface gigabyte0/0._numero du vlan_**

il ne faut pas donner d'adresse IP à l'interface physique mais l'activer: **no shutdown**

les interfaces virtuelles ce comportent exactement comme une interface physique, la configuration est donc la même:
**ip address _adresse ip de l'interface_ _masque correspondant_**

il faut cependant indiquer que les trames provenant des vlan seront encapsulé pour pouvoir communiquer sur le réseau:
**encapsulation dot1Q _numero du vlan_**

## Routage dynamique (RIP)

le routage dynamique permet de donner aux routeurs les routes pour communiquer avec d'autre réseau. il offre l'avantage que les routeurs communiqueront entre eux les réseaux auxquels ils ont accés et trouverons seul la route qui convient.

RIP fonctionne sur la base des adresses IP classful, en regardant la configuration des réseaux: **show running-config** ou **show ip rip database** on remarquera que les adresses réseaux que nous lui avons donné ne respecte pas forcément le sous-découpage que nous avons réalisé. Cependant en regardant les **routes** grâce à la commande: **show ip route** les adresses réseau ainsi que leur masque correspondant sont indiqué correctement.

RIP ce base sur **un algorithme de vecteur de distance**, c'est à dire qu'il compte le nombre de station qu'un paquet doit parcourir pour arrivé à destination. Il ne prend donc pas en compte la vitesse ou le type de câble utilisé. 

pour une configuration avec le protocole RIP:
**router rip** => entré dans la configuration du protocole
**version 2** => nous utiliseront la version 2 qui est plus sécurisé et supporte le VLSM
**network _address IP du réseau_** => indiquer à RIP les réseaux auxquels ce routeur à accès

seul les routes auxquels le routeur à **directement accès** doivent être indiqué à RIP, si il y a plusieurs routeur dans la topologie, il faudra indiqué à chacun les réseaux auxquels ils sont connecté.

si une route par défaut dois être configuré (pour sortir sur internet par exemple) il faudra rajouter sur le routeur **frontalier:** 
**default-information originate** 

cette commande informe RIP qu'une route par default est configuré en statique et qu'il dois la transmettre aux autres routeurs

## Routage dynamique (OSPF)

tout comme RIP; OSPF fourni le même service mais fonctionne de manière différente:

OSPF ce base sur un algorithme de plus court chemin (dijkstra) pour choisir la meilleur voie à parcourir. Contrairement à RIP; OSPF prend en compte la vitesse d'un câble et l'utilise comme **poids**; addition tout les poids jusqu'à destination. le chemin qui à le poids le plus faible est sélectionné pour être la route qui sera mise dans la table de routage.

 OSPF fonctionne également avec un système **d'area** , étant donner que OSPF est très bavards (il envoi beaucoup de paquet sur le réseau pour avoir une tolérance de panne élevé) sur un réseau de taille conséquente cela peut affecté la bande passante. **les zone permettent donc de limité les communication.** 

**l'area 0** est celle qui collecte toute les informations de toute les zones pour permettre la communication inter-area. Il est donc important de la placé de manière stratégique. 

configuration du routage avec OSPF:

**router ospf 1** => entré dans la configuration 1 du routage OSPF
**network _adresse-du-reseau_ _masque-inversé_ area _numero-de-zone_** => ajout d'un réseau

tout comme pour RIP, **default-information originate** permet d'informer OSPF de communiquer une route par défaut configuré en statique.

## routage dynamique (EIGRP)

EIGRP est un protocole de routage dynamique **propriétaire CISCO** il est donc impossible de le déployé si un appareil de la topologie n'est pas cisco.

il ce configure simplement:
- **router EIGRP 1**
- **network *ip reseau connecté directement sur le routeur**
la valeur **1** indiqué dans la première commande à une grande importance. toute la topologie doit avoir la même valeur ou ils ne communiqueront pas ensemble.

EIGRP fonctionne avec un calcul plus précis que OSPF, il prend en compte la **bande passante** et le **délai des interfaces** pour calculé la route optimal pour communiqué.
Il est possible d'ajouter des paramètre au calcul tel que: la charge ou le MTU.

En créant ça table de routage, EIGRP fabrique une second table contenant des routes en cas de défaillance d'une route, il n'a pas à re calculé toute les routes. Cela lui permet d'être le protocole de routage le plus efficace de tous.

## Routage dynamique inter protocol

dans une infrastructure il est possible d'avoir deux protocoles de routage différents et de faire transité les informations entre eux. pour cela dans la configuration des protocoles il faut ajouter la commande:

**redistribute _nom-du-protocole**

## access-list (ACL)

Une access-list permet de mettre des restrictions sur les interfaces des routeurs pour empêcher un réseau, une machine ou un protocole de communiqué.

il existe deux catégorie d'access-list:
- **les access-list stantard:**
	- elles ne permettent que de simple restrictions en prenant en paramètre que l'adresse ip source (machine ou réseau)
- **les access-list extented:**
	- elles permettent beaucoup plus de précision en prenant en paramètre l'adresse source, l'adresse de destination, le protocole et le port. selon le protocole spécifié des paramètres plus précis peuvent être ajouté (un echo reply peut être bloqué mais pas un request par exemple)

il est possible de crée des acces-list par numéro ou par nom:
- **par numéro:**
	- de 1 à 99 le routeur interprétera la liste comme une liste standard
	-  de 100 à 199 le routeur interprétera la liste comme une liste standard
- **par nom:**
	- un nom peut être donné en spécifiant dans la commande le type de liste que nous voulons. le routeur assignera en interne un numéro correspondant au type de liste demandé.

### access-list standard:
pour crée une access-list standard:
**access-list <numéro de 1 à 99> deny/permit <any/host/adresse réseau source>**
**ip access-list standard *nom de la liste* deny/permit <any/host/adresse réseau source>**
une fois l'access-list crée il faut l'attribué à une interface.
*accéder à la configuration de l'interface désiré*
**ip access-group <numéro/nom> <in/out>**

in et out indique si la list dois traité les flux entrant (in) dans le routeur ou sortant (out)

### access-list extended

pour crée une access-list étendu:
**access-list <numéro de 100 à 199> deny/permit *nom du protocole* *any/host/adresse reseau source* *masque INVERSE correspondant* *any/host/adresse reseau destination* *masque INVERSE correspondant * <eq/neq/gt/lt>*numéro de port utilisé par le protocole/type de paquet*

#### ATTENTION 

les acces-list acceptent plusieurs deny/permit mais il faut les ajouter dans la liste **de la plus précise à la plus général**; sinon les restrictions plus loin dans la liste ne seront pas prise en compte. 

si une liste doit être modifié avec une instruction plus précise, il faudra supprimé la liste et la refaire au complet. Il est recommandé d'utilisé un bloc-note pour garder les listes et pouvoir les copier/coller. 

**deux listes** peuvent être appliqué par interface:
	- UNE SEULE en entré
	- UNE SEULE en sortie

eq 	= 		equal
neq 	= 		non equal
lt 		= 		less than
gt 	= 		greater than


## netwwork address translation (NAT)

il existe trois type de NAT:
- le nat **statique**
- le nat **dynamique**
- le nat **port address translation (PAT)**

le NAT permet de faire la traduction entre une adresse IP **privée** en une adresse IP **publique** pour permettre au devices dans le réseau privé de comuniqué sur internet.

**le nat statique:**
- il permet d'attribué une adresse privé à une adresse publique. utilisés pour mettre au monde d'accéder à un serveur par exemple.
- **ip nat inside source static <adresse_privée> <adresse_publique>**
- sur l'interface d'entré dans le routeur on précise **ip nat inside** pour indiqué au routeur que l'adresse à traduire arrivera de ce port.
- sur l'interface de sortie (vers le réseau WAN) on indique: **ip nat outside** pour dire au routeur sur quel port envoyé l'adresse traduite.

**le nat dynamique:**
le dynamique permet d'associer plusieurs adresse privé à une ou plusieurs adresse publique. cependant si toute les adresse ip publique sont occupé la machine suivante qui demande accés à internet ne pourra pas obtenir d'adresse tant qu'un autre appareil ne ce sera pas déconnecté.

- il faut d'abord configuré une access-list contenant les réseaux qui ont le droit de sortir:
	- **access-list <numéro ou nom> permit  <ip_source> *masque***
	- il faut ensuite définir la ou les addresses ip publique à notre disposition:
	- **ip nat pool *nom du pool* *ip de départ* *ip de fin* netmask *masque***
	- définir la traduction:
	- **ip nat inside source list <nom de l'access-list> pool *nom du pool***
- dans le cas ou nous avons plus d'adresse privé ayant besoin d'accéder à internet (on parle de nat avec surcharge) on ajoute **overload** à la fin de la dernière commande.

**le pat:**
le pat permet de déplacé le problème en attribuant à chaque demande de traduction un numéro de port aléatoire et unique qui permettra d'identifié quel est la machine qui souhaite communiqué.
- tout comme le nat dynamique il faut crée une access-list des réseaux autorisé à sortir:
	-  **access-list <numéro ou nom> permit  <ip_source> *masque***
- puis ip nat inside source list <numéro de la liste> interface <nom de l'interface de sortie> overload
- si nous avons plusieurs addresses publique à disposition il faudra crée un pool et l'attribué:
	- **ip nat pool *nom du pool* *ip de départ* *ip de fin* netmask *masque***
	- **ip nat inside source list <nom de l'access-list> pool *nom du pool***

tout comme en mode static et dynamique il faudra préciser dans les interfaces **ip nat inside** ou **ip nat outside** selon si il sagira d'un réseau entrant ou sortant.