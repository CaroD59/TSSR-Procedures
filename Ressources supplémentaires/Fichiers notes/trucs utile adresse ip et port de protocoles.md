## classless et classful

Les adresses IP sont divisé en 5 class mais seulement 3 sont utilisable: 
| classes | description | 
|-- |--|
| A | un seul octet pour la partie réseau |
| B | 2 octet pour la partie réseau |
| C | 3 octets pour la partie réseau |
| D | réservé aux protocols  |
| E | réservé aux test pour l'ICANN |

la classe A vas de 1.0.0.0 à 126.0.0.0
la classe B vas de 128.0.0.0 à 191.0.0.0
La classe C vas de 192.0.0.0 à 223.0.0.0
la classe D vas de 224.0.0.0 à 239.0.0.0
la classe E vas de 240.0.0.0 à 255.0.0.0

**il existe cependant certaines adresse spécial:**
0.0.0.0 => signifie *ce réseau*
255.255.255.255 => est l'adresse de broadcast
127.0.0.1 => adresse de loopback (utilisé pour pointé vers moi-même
169.254.0.0 jusque 169.254.255.255 => adresse APIPA, l'appareil ce donnera cette adresse si il n'arrive pas à en obtenir une par DHCP

**adresse privé:**
dans la classe A, l'adresse: 10.0.0.0 jusque 10.255.255.255 est à usage privé
en classe B, les adresses 172.16.0.0 jusque 172.31.255.255 sont à usage privé
en classe C, les adresse de 192.168.0.0 à 192.168.255.255 sont à usage privé

##liste des protocoles les plus utilisés avec leur ports:

FTP					TCP							20/21
SSH					TCP							22
Telnet				TCP							23
SMTP				TCP							25
DNS				TCP/UDP				53
TFTP				UDP						6
HTTP				TCP							80
HTTP				TCP							443
POP3				TCP							110
IMAP				TCP							143