## ISCSI sur un serveur Windows

*au préalable veillez à avoir un disque (virtuel ou physique) disponible et en ligne*

par défaut, les disques sur un serveur windows sont en mode "hors ligne" suite à une politique de sécurité; suivez les étapes ci-dessous pour la modifié:

		- ouvrez un terminal (Win+R et taper "cmd")
		- taper la commande "Diskpart" pour entré dans la configuration des disques
		- taper la commande "san" pour connaitre la stratégie actuellement en place
		- taper la commande "san policy=onlineall" pour modifié la stratégie et la définir au mode "en ligne" par défaut (assuré vous qu'un message de confirmation vous es donné)
		- pensé également à initialisé, formaté et partitionné le disque avant de continuer

#### ajout des roles ISCI
	- dans le gestionnaire de serveur, ajouter un role 
	- cherché "service de fichier et de stockage" et le dérouler.
	- dérouler "service de fichier et ISCSI"
	- ajouter les roles: 
		- fournisseur de stockage cible ISCSI 
		- serveur cible ISCSI
	- installé les services
	

#### configuration de ISCI
	
	- dans le menu vertical sur la gauche de la fenetre principale cherché: "service de fichier et de stockage"
	- puis aller dans l'onglet "ISCSI"
	- démarrer l'assistant ISCSI pour la création d'un nouveau disque
	- selectionné le disque à désigner comme cible
	- donner lui un nom et une description
	- définissez la taille à partager
	- si vous avez déjà une cible ISCSI vous pouvez lui attribué ou en crée une nouvelle
	- dans le cas d'une nouvelle cible, donner lui un nom et une description
	- ajouter les initiateurs pouvant accéder à la cible
	- dans la pop-up sélectionné dans le menu déroulant "IP" et indiqué l'adresse IP de la ou les machine ayant accés à cette cible
	- dans un cadre professionnel, activé le protocol CHAP pour augmenter la sécurité 
	- vérifié que les informations sont correcte et cliquer sur crée

#### ajouter le nouveau disque a ESXI depuis VCenter

	- sélectionner le serveur ESXI souhaité, aller dans l'onglet "configuré"
	- puis cliqué sur "ajoute un adaptateur "
	- "ajouter un adaptateur logiciel"
	- sélectionne l'adaptateur crée, dans la fenetre qui s'ouvre en bas aller dans le menu "découverte dynamique"
	- clqiuer sur "ajouter" puis indiqué l'adresse IP du serveur sur lequel ce trouve le disque
	- une fois fait dans la fenetre supérieur, cliquer sur "réanalysé le stockage"

#### création d'une nouvelle banque de donnée

	- dans l'interface de Vcenter
	- dans l'arborescence faire un clic droit sur l'espace de stockage principale 
	- dans l'onglet "stockage" sélectionner "nouvelle banque de données"
	- sélectionner "NFS" uniquement si vous devez crée des fichiers partagé sur ce disque entre Windows, Linux et/ou MAC; sinon il est préférable d'utilisé "VMFS" pour éviter des conflict avec VMWare
	- renomé votre banque de données et sélectionné l'hote ESXI (peut importe lequel vous choissisez, elle montera sur tous)
	- partitionné le disque comme vous en avez besoin

pour vérifier l'installation rendez-vous sur vos serveur ESXI et rendez-vous dans l'onglet "banque de donné"