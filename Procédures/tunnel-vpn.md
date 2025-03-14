# Création d'un tunnel VPN avec IPSec sur PfSense

### Explication
Un tunnel VPN permet une connexion à distance sécurisée entre deux sites (appelé **site-to-site**).

### Procédure
Sur **PfSense**, rendez-vous dans l'onglet **VPN**, puis **IPsec** et cliquez sur **Add p1**.

#### Description
> La description est à usage indicatif. Généralement, on indique la destination du tunnel.

#### Disabled
> Décochez cette case pour que le tunnel soit opérationnel une fois la configuration terminée.

#### Key Exchange Version
> Spécifiez la version que vous souhaitez utiliser. **IKEv2** est préférable car plus sécurisé, mais assurez-vous que le pare-feu de l'autre côté supporte cette version. Sinon, utilisez **IKEv1**.

#### Internet Protocol
> Par défaut, **IPv4** est utilisé. Si vous souhaitez utiliser **IPv6**, vous pouvez le spécifier ici.

#### Interface
> Indiquez l'interface sur laquelle vous allez mettre en place votre tunnel (généralement la **WAN**). Référez-vous à votre infrastructure.

#### Remove Gateway
> Entrez l'adresse IP de l'interface de **Destination**.

---

### Partie suivante : Authentification de la phase 1 de IPsec
Nous garderons les informations par défaut pour simplifier. Vous pouvez changer la méthode d'authentification à votre guise, mais faites attention à bien comprendre les implications des modifications.

#### Authentication Method
> Laissez **Mutual PSK**.

#### My Identifier
> Identifiant utilisé pour l'authentification.

#### Peer Identifier
> Par défaut, **Peer IP address**.

#### Pre-shared Key
> Utilisez une clé sécurisée (au minimum 10 caractères, incluant minuscules, majuscules, chiffres et symboles).  
> Vous pouvez demander à **PfSense** de générer une clé pour vous en cliquant sur **Generate new pre-shared key**.
> 
> **ATTENTION** : Vous devrez entrer **exactement cette clé** lors de la configuration sur l'autre pare-feu. Il est conseillé de la copier et de la coller dans un fichier texte pour éviter toute erreur.

---

### Partie suivante : Cryptage de la phase 1 de IPsec

#### Encryption Algorithm
> Utilisez la méthode de cryptage **AES** avec une taille de **256 bits**.

#### Hash Algorithm
> Si les deux pare-feu supportent le hash **SHA256**, utilisez-le. Sinon, optez pour la méthode de hash la plus efficace disponible sur les deux appareils.

#### DH Group
> La valeur par défaut **14 (2048 bits)** est correcte. Vous pouvez utiliser une valeur plus élevée, mais cela consommera davantage de ressources CPU.

---

### Expiration et remplacement
Cette section contrôle la fréquence et la méthode de négociation de la phase 1.

#### Life Time
> La valeur par défaut **28800** secondes (8 heures) est suffisante.

Les autres valeurs (**Rekey Time**, **Reauth Time**, **Rand Time**) doivent être laissées par défaut, car un calcul est effectué automatiquement et cette configuration est correcte.

### Section avancée
La section avancée contient des paramètres intéressants :

#### Child SA Close Action
> Sélectionnez **Restart/Reconnect** pour reconnecter la phase 2 si elle est déconnectée.

#### Dead Peer Detection
> Cochez cette case (si ce n'est pas déjà fait) et laissez les valeurs par défaut. Ce paramètre permet de savoir si l'hôte distant est toujours "en vie".

Cliquez sur **Save** et la phase 1 est maintenant configurée.

---

## Configuration de la phase 2

Cliquez sur **"Show Phase 2 Entries"**, puis **"Add p2"**.

#### Description
> Comme pour la phase 1, vous pouvez indiquer ici les réseaux connectés.

#### Disable
> Cochez cette case pour désactiver la phase 2 si nécessaire.

#### Mode
> Nous configurons un tunnel **IPv4**, laissez donc la valeur par défaut.

#### Local Network
> Vous pouvez indiquer manuellement l'adresse IP de votre réseau local. Cependant, il est recommandé de laisser la valeur **LAN subnet**. Cela permet au tunnel de continuer à fonctionner même si l'adresse change à l'avenir.

#### NAT/BINAT
> Si du NAT est en place sur votre réseau, configurez-le ici. Sinon, laissez-le à **none**.

#### Remote Network
> Indiquez ici le réseau de destination du tunnel.

---

### Cryptage du trafic entre les deux sites

#### Protocol
> Définissez **ESP** pour le cryptage.

#### Encryption Algorithm & Hash Algorithm
> Les meilleures pratiques sont d'utiliser un chiffrement authentifié (AEAD), tel que **AES-GCM**. Cependant, vérifiez que les deux côtés du tunnel le supportent.

Si c'est le cas :
> Sélectionnez **AES256-GCM** avec une taille de **128 bits** et ne sélectionnez pas d'algorithme de hash.

Si ce n'est pas supporté :
> Utilisez **AES avec une taille de 256 bits** et un algorithme de hash comme **SHA256**. Si cela n'est pas supporté des deux côtés, utilisez les algorithmes les plus sécurisés à votre disposition.

#### PFS (Perfect Forward Secrecy)
> Ce paramètre est optionnel mais permet de protéger la clé contre certaines attaques. Vous pouvez laisser la valeur par défaut.

#### Life Time
> Comme pour la phase 1, laissez les valeurs par défaut.

Vérifiez les informations renseignées, puis sauvegardez et appliquez les changements.

---

## Mise en place des règles de pare-feu pour le tunnel

Rendez-vous dans l'onglet **Pare-feu**, puis dans **Règles**. Sélectionnez ensuite la section **IPsec**.

Vous pouvez définir les règles de trafic que vous souhaitez, mais assurez-vous que les adresses **source** et **destination** correspondent aux IP de vos deux sites.

---

### Félicitations, votre tunnel VPN site-to-site est maintenant configuré !