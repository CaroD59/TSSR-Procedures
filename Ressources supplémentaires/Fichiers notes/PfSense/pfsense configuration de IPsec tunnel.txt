## création d'un tunnel VPN avec IPSec sur PfSense

### Explication
un tunnel vpn permet la connection à distance de manière sécurisé entre deux sites (dit site-to-site)

### Procédure
sur PfSense, ce rendre dans l'onglet "VPN"  puis IPsec
cliquer sur "Add p1"

#### Description
>la description est à usage indicatif, généralement on indique la destination du tunnel

#### Disabled
>décocher cette case pour que le tunnel soit opérationnel une fois la configuration terminé

#### Key Exchange Version
>spécifier que version vous souhaitez utilisé, IKEv2 est préférable car plus sécurisé mais assuré que le pare-feu de l'autre coté supporte cette version, sinon utilisé IKEv1

#### Internet Protocol
>généralement IPv4 est utilisé mais si vous souhaitez utilisé IPv6, c'est ici que vous le spécifié

#### Interface
>indiquer l'interface sur laquelle vous allez mettre en place votre tunnel (en général la WAN) référez vous à votre infrastructure

#### Remove Gateway
>entré l'adresse IP de l'interface de **Destination**

---
### La prochaine partie concerne l'authentification de la phase1 de IPsec
nous garderons les informations par défaut pour une question de simplicité, vous pouvez changer la méthode d'authentification à votre guise mais faites attention à savoir ce que vous faites

#### Authentification Method
>laissez "Mutual PSK"

#### My Identifier
> identifiant utilisé pour l'authentification

#### Peer Identifier
> par default "Peer IP address"

#### Pre-shared Key
> utilisé une clé sure; au minimum 10 caractère, minuscules, majuscules, chiffres et symboles.
> Vous pouvez demandé à PfSense de généré une clef pour vous en cliquant sur "generate new pre-shared key"
> 
> **ATTENTION** vous devrez rentré **exactement cette clef** lors de la configuration sur l'autre pare-feu, je vous conseil de la copier-coller dans un bloc note

---
### La partie suivante concerne l'encryption de la phase1 de IPsec

#### Encryption Algorithm
> utilisé la méthode d'encryption **AES** avec une taille de **256bits**

#### Hash algorithm
> si les deux pare-feu support le hash "SHA256" utilisé le, sinon utilisé la méthode de hash la plus efficace disponible sur les deux appareils

#### DH Group
>la valeur pas défaut 14(2048) est OK, vous pouvez utilisé une valeur plus élevé mais cela consomera plus sur le CPU

### L'expiration et le remplacement
cette section control de quel façon et à quel fréquence la phase 1 sera négocier à nouveau

#### Life Time
> la valeur par défaut 28800 est suffisante

Les autres valeur (**Rekey Time**, **Reauth Time**, **Rand Time**) devrait être laissé ainsi car un calcul est à faire; par défaut, cette configuration est correcte.

### La section avancé
la section avancé contient quelques paramètres intéressant:

#### Child SA close action
> sélectionné "restart/reconnect" pour reconnecter la phase 2 si elle est déconnecter

#### Dead Peer Detection
> cocher la case si pas fait et laisser les valeur par défaut; ce paramètre permet de savoir si l'hote distant est toujours "en vie"

cliquer sur **save** et la phase1 est maintenant configuré

## configuration de la phase 2

cliquer sur **"show phase 2 entries"** puis **"add p2"**

#### Description
> comme pour la phase , on indique habituellement ici les réseaux connecté

#### Disable
>désactive la phase si cocher

#### Mode
> Nous configurons un tunnel IPv4, laissez donc la valeur par défaut

#### local network
> vous pouvez indiquez manuellement l'adresse IP de votre réseau local; cependant on recommande de laisser "LAN subnet" dans le cas ou l'adresse changerez dans le futur, le tunnel continuerez de fonctionner

#### NAT/BINAT
>Si du NAT est en place sur votre réseau, configuré le ici; sinon laissez à "none"

#### Remote Network
>indiqué ici le réseau de destination du tunnel

---
### la section suivante concerne le cryptage du trafique entre les deux site

#### Protocol
>défini à ESP pour le cryptage

#### Encryption Algorithm & hash algorithm
> Les meilleurs pratique sont d'utilisé un AEAD (chiffrement authentifié) tel que AES-CGM; cependant vérifié que les deux coté du tunnel le supporte.

si oui:
> sélectionné AES256-CGM avec une taille de 128 bits et ne sélectionné pas d'algorithme de hash

si non:
>utilisé AES avec une taille de 256 et un algorithme de hash tel que SHA256. Si cela n'est pas supporté par les deux coté, utilisé les algorithmes les plus fort à votre disposition

#### PFS (Perfect Forward Secrecy)
>ce paramètre est optionnel mais permet de protéger la clef de certaines attaques, vous pouvez laissez la valeur par défaut

#### Life Time
> comme pour la phase1, laissez les valeurs par défaut

vérifié les informations renseigné puis sauvegardé et appliqué les changements.

## mise en place des règles de pare-feu pour le tunnel

rendez-vous dans l'onglet pare-feu puis règles, et enfin dans la partie "IPsec"

vous pouvez indiqué les règles de trafique que vous voulez, cependant assuré vous que les adresses source/ destination correspondent au IP de vos deux sites.

FELICITATION VOTRE SITE A EST CONFIGURE !

## Configuration du site B 

la configuration du site B sera sensiblement la même que sur le site A mais avec tout de même quelques différences.

#### Description
>indiquer ici ou pointe ce tunnel

#### Remote Gateway
>renseigner l'adresse ip de l'interface à laquelle vous voulez vous connecté (celle du site A par exemple)

#### authentification
>dans le champ réservé à la clef d'authentification, entré celle que vous as générer pfsense lors de la configuration du site A

#### life time
> dans l'idéal augmenter les deux 10% minimum

#### Child SA Close Action
> mettez ce paramètre à "Close connection and clear SA"; c'est le site A qui s'occupera d'initialiser la connection à nouveau

sauvegarder et appliquer les changements. Puis comme pour le site A, rendez-vous dans les règles de votre pare-feu puis dans l'onglet IPsec pour définir les règles en vigeur dans le tunnel.

## Vérification

vous pouvez vérifier le statut de votre tunnel dans l'onglet: status > IPsec
Si votre configuration c'est bien passé, vous devriez voir apparaitre "established" si c'est pas le cas, il est possible que l'initialisation ne ce fasse que quand du trafique veux traversé le tunnel. Pour cela un bouton "Connect VPN" est disponible. 

Si le bouton n'apparait pas, tenter un ping depuis le site configuré comme initiateur vers le second site.

FELICITATION VOTRE TUNNEL EST FONCTIONNEL !