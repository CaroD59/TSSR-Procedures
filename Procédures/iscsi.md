## iSCSI sur un serveur Windows

*Au préalable, veillez à avoir un disque (virtuel ou physique) disponible et en ligne.*

Par défaut, les disques sur un serveur Windows sont en mode "hors ligne" en raison d'une politique de sécurité. Suivez les étapes ci-dessous pour la modifier :

- Ouvrez un terminal (Win+R et tapez "cmd").
- Tapez la commande `Diskpart` pour entrer dans la configuration des disques.
- Tapez la commande `san` pour connaître la stratégie actuellement en place.
- Tapez la commande `san policy=onlineall` pour modifier la stratégie et la définir en mode "en ligne" par défaut (assurez-vous qu'un message de confirmation vous soit donné).
- Pensez également à initialiser, formater et partitionner le disque avant de continuer.

#### Ajout des rôles iSCSI

- Dans le gestionnaire de serveur, ajoutez un rôle.
- Cherchez "Service de fichiers et de stockage" et déroulez-le.
- Déroulez "Service de fichiers et iSCSI".
- Ajoutez les rôles suivants :
  - Fournisseur de stockage cible iSCSI
  - Serveur cible iSCSI
- Installez les services.

#### Configuration de iSCSI

- Dans le menu vertical à gauche de la fenêtre principale, cherchez : "Service de fichiers et de stockage".
- Allez ensuite dans l'onglet "iSCSI".
- Démarrez l'assistant iSCSI pour la création d'un nouveau disque.
- Sélectionnez le disque à désigner comme cible.
- Donnez-lui un nom et une description.
- Définissez la taille à partager.
- Si vous avez déjà une cible iSCSI, vous pouvez lui attribuer ou en créer une nouvelle.
- Dans le cas d'une nouvelle cible, donnez-lui un nom et une description.
- Ajoutez les initiateurs pouvant accéder à la cible.
- Dans la pop-up, sélectionnez dans le menu déroulant "IP" et indiquez l'adresse IP de la ou des machines ayant accès à cette cible.
- Dans un cadre professionnel, activez le protocole CHAP pour augmenter la sécurité.
- Vérifiez que les informations sont correctes et cliquez sur "Créer".

#### Ajouter le nouveau disque à ESXi depuis vCenter

- Sélectionnez le serveur ESXi souhaité, puis allez dans l'onglet "Configurer".
- Cliquez sur "Ajouter un adaptateur".
- Sélectionnez "Ajouter un adaptateur logiciel".
- Sélectionnez l'adaptateur créé, puis dans la fenêtre qui s'ouvre en bas, allez dans le menu "Découverte dynamique".
- Cliquez sur "Ajouter", puis indiquez l'adresse IP du serveur où se trouve le disque.
- Une fois fait, dans la fenêtre supérieure, cliquez sur "Réanalyser le stockage".

#### Création d'une nouvelle banque de données

- Dans l'interface de vCenter.
- Dans l'arborescence, faites un clic droit sur l'espace de stockage principal.
- Dans l'onglet "Stockage", sélectionnez "Nouvelle banque de données".
- Sélectionnez "NFS" uniquement si vous devez créer des fichiers partagés sur ce disque entre Windows, Linux et/ou macOS. Sinon, il est préférable d'utiliser "VMFS" pour éviter des conflits avec VMware.
- Renommez votre banque de données et sélectionnez l'hôte ESXi (peu importe lequel vous choisissez, elle montera sur tous).
- Partitionnez le disque comme vous en avez besoin.

Pour vérifier l'installation, rendez-vous sur vos serveurs ESXi et allez dans l'onglet "Banque de données".
