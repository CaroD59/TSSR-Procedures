# pare-feu PFsense

### marche à suivre lors de la création de règles
lorsque l'on veux autorisé des connections, l'idéal et d'utilisé wireshark (ou autre sniffer) et d'autorisé petit à petit les ports et protocol nécessaire au service que l'on souhaite mettre en place. 
L'ajout de description dans les allias, et dans les règles mises en place est primordial pour facilité la compréhension global.
Les allias sont très utiles également pour limiter le nombre de règles faisant références a un même protocole

#### autorisation d'un réseau à accéder à un serveur web présent dans une DMZ 

la démarche étant sensiblement la même pour autorisé à accéder à internet, elle ne sera pas détailler ici.

pour l'accès à internet, 3 protocoles sont utilisé:
- DNS 	(port 53)
- http		(port 80)
- https		(port 443)

créé donc un allias contenant les ports 80 et 443 que vous pourrez nommé: "internet"
puis crée deux règles:
- une contenant l'autorisation du poste / réseau en TCP/UDP vers votre serveur DNS
- une seconde contenant l'autorisation du poste / réseau vers les ports de l'allias précédemment crée (aka internet)

#### autorisation nécessaire pour permettre de rejoindre un domaine active directory
il y aura plusieurs protocoles à autorisé pour ce service:
- Kerberos
	- port 88 TCP & UDP
- DNS
	-  port 53 TCP & UDP
- RPC
	- port 135 TCP & UDP
- RPC Dynamic
	- de 49135 à 65535 TCP
- LDAP
	- port 389 TCP & UDP
- LDAPS
	- port 636 TCP
- SMB
	- port 137,138 UDP ?
	- port 139,445 TCP ?
- ICMP

il est important d'ajouter en dernière ligne: deny	any	any pour empêcher tout autre protocole de passer après ceux qui ont était autorisé.