# Mise en place de la Haute Disponibilité sur PfSense

## Introduction
Pour mettre en place la haute disponibilité (HA) sur PfSense, il faut commencer par créer une **adresse IP virtuelle** qui sera utilisée à tour de rôle par les serveurs PfSense pour assurer la continuité des services. Cela est réalisé grâce au protocole CARP. 
Il y a deux autres protocoles que nous utiliserons : pfSync et XML-RPC.

- **pfSync** : permet la synchronisation entre les serveurs PfSense et la transmission des connexions en cours.
- **XML-RPC** : permet la transmission de données entre les serveurs. Il doit être mis en place sur la même interface que pfSync.

### 1. Création de l'adresse IP virtuelle (VIP)

Rendez-vous dans l'onglet **Firewall** puis dans **Virtual IPs** et ajoutez une nouvelle IP virtuelle.

#### Type
> Nous utiliserons ici le type **CARP**.

#### Interface
> Sélectionnez l'interface sur laquelle vous souhaitez déployer votre VIP.

#### Address
> Entrez l'adresse IP que vous souhaitez utiliser comme virtuelle. 
> **ATTENTION** : Utilisez une adresse **VALIDE** sur le réseau de votre interface et qui **n'est pas utilisée** par une autre carte virtuelle.

#### Virtual IP Password
> Mot de passe permettant de sécuriser les échanges entre les hôtes qui partageront cette adresse.

#### VHID Group (Virtual Host Identifier)
> Permet aux hôtes de s'identifier. Un hôte peut faire partie de plusieurs groupes. Nous laisserons la valeur par défaut.

#### Advertising Frequency
> La valeur du champ "skew" permet aux hôtes de savoir qui est le serveur primaire et qui est le secondaire. Plus la valeur est élevée, plus l'hôte qui l'a est secondaire. Nous laisserons la valeur par défaut.

Une fois votre adresse configurée sur l'interface **WAN**, effectuez la même opération sur l'interface **LAN**. Enfin, réalisez la même configuration sur le PfSense de secours, en pensant bien à changer la valeur du champ **skew** à 1. Faites attention au **VHID Group** : pour chaque VIP, il doit être identique sur les deux firewalls.

### 2. Vérification

Pour vérifier que vos VIP sont bien configurées et prises en compte, rendez-vous dans l'onglet **Status** > **CARP (failover)**.

Si tout s'est bien passé, vos VIP sont bien configurées avec **MASTER** sur votre PfSense principal, et **BACKUP** sur votre PfSense secondaire.

## 3. Forçage des VIP

Vos VIP sont configurées mais pas encore utilisées. Il faut donc indiquer à PfSense d'utiliser ces VIP.

#### Utilisation des VIP
> Rendez-vous dans l'onglet **Firewall** > **NAT**.
> Changez le mode utilisé par le NAT pour : **Hybrid Outbound NAT rule generation**.
> Cliquez sur **Save**.

À présent, nous allons créer une nouvelle règle pour utiliser les VIP.

#### Paramètres de la règle NAT

- **Disable** : La case "Disable" désactivera la règle une fois créée. Laissez cette case décochée.
- **Do not NAT** : Cocher cette case permet de désactiver le NAT pour cette règle. Nous la laisserons décochée.
- **Interface** : L'interface sur laquelle nous allons appliquer notre règle de NAT. Ici, l'interface **WAN**.
- **Protocol** : Les protocoles concernés par cette règle de NAT. Ici, "any".
- **Source** : Le réseau source, dans notre cas, le réseau local.
- **Destination** : Le réseau de destination, dans notre cas, nous mettrons "any".
- **Address** : L'adresse IP à utiliser pour cette règle, c'est ici que nous renseignons la VIP créée auparavant.
- **Port** : Spécifie un port pour cette règle de NAT. Nous le laisserons vide.
- **No XMLRPC Sync** : Cocher cette case pour ne pas copier les paramètres sur le serveur secondaire. Nous laisserons cette case décochée.
- **Description** : Un champ informatif pour décrire ce que fait cette règle.

## 4. Configuration de la Haute Disponibilité

Rendez-vous dans **System** > **High Avail. Sync**.

### 4.1. State Synchronization settings (PfSync)

- **Synchronize states** : Cochez cette case pour activer pfSync.
- **Synchronize interface** : Si vous avez mis en place une interface supplémentaire pour synchroniser les PfSense, choisissez-la. Ici, nous sélectionnerons **LAN**.
- **Filter Host ID** : Identifiant utilisé par le protocole PfSync. Laissez ce champ vide pour que PfSense choisisse automatiquement.
- **PfSync synchronize Peer IP** : Entrez l'adresse IP du serveur PfSense de secours. **ATTENTION** : Cette adresse doit correspondre à l'interface de synchronisation choisie précédemment. Si aucune adresse n'est définie, PfSense diffusera en multicast (ce qui constitue une faille de sécurité). 
> **IMPORTANT** : Il faudra entrer l'adresse IP du PfSense **principal** à cet endroit sur **le PfSense secondaire**.

### 4.2. Configuration Synchronization settings (XMLRPC Sync)

- **Synchronize Config to IP** : Entrez l'adresse IP à laquelle le PfSense enverra les données (ici, l'IP du PfSense de secours).
- **Remote System Username** : Entrez le nom d'utilisateur de l'administrateur du PfSense de secours.
- **Remote System Password** : Entrez le mot de passe correspondant. (Si vous ne l'avez pas changé, **faites-le**).
- **Synchronize admin** : Synchronise les comptes administrateur entre les deux PfSense.
- **Select options to sync** : Permet de sélectionner les données des services que vous souhaitez synchroniser. Ici, nous sélectionnons **tout**.

Sauvegardez et c'est configuré ! :)

## 5. Autorisations sur le pare-feu

Rendez-vous dans l'onglet **Firewall** > **Rules**. Nous avons deux règles à ajouter sur l'interface sur laquelle nos PfSense vont communiquer :
- Le flux pour la synchronisation **XMLRPC** (port 443).
- Le flux pour la synchronisation **PfSync**.

Vous aurez besoin de créer deux règles pour chaque adresse IP. Si vous souhaitez suivre les bonnes pratiques, créez un **alias** (onglet **Firewall** > **Alias**) contenant les deux adresses IP des interfaces de synchronisation.

### 5.1. Création d'une règle pour autoriser XMLRPC

- **Action** : Pass
- **Interface** : Indiquer l'interface sur laquelle nous allons mettre en place la règle. Ici **LAN**.
- **Address Family** : Indiquer si cette règle affecte l'IPv4 ou IPv6. Pour nous ce sera **IPv4**.
- **Protocol** : Indiquer le protocole concerné par cette règle. Dans notre cas, ce sera **TCP**.
- **Source** : Indiquer l'adresse source. Renseigner l'alias créé ou une des deux adresses IP des interfaces de synchronisation du PfSense secondaire (vous devrez créer une seconde règle identique avec l'autre adresse IP).
- **Destination** : Indiquer l'adresse de destination. Dans notre cas, "**My firewall(self)**".
- **Destination Port Range** : Choisir le protocole **HTTPS** (port 443).
- **Description** : Champ informatif non obligatoire pour décrire cette règle.

### 5.2. Création de la règle pour autoriser PFSYNC

- **Action** : Pass
- **Interface** : Indiquer l'interface sur laquelle nous allons mettre en place la règle. Ici **LAN**.
- **Address Family** : Indiquer si cette règle affecte l'IPv4 ou IPv6. Pour nous ce sera **IPv4**.
- **Protocol** : Indiquer le protocole concerné par cette règle. Dans notre cas, ce sera **PFSYNC**.
- **Source** : Indiquer l'adresse source. Renseigner l'alias créé ou une des deux adresses IP des interfaces de synchronisation du PfSense secondaire (vous devrez créer une seconde règle identique avec l'autre adresse IP).
- **Destination** : Indiquer l'adresse de destination. Dans notre cas, "**My firewall(self)**".
- **Description** : Champ informatif non obligatoire pour décrire cette règle.