# GPO

information général:
- les dossiers accessible sur le réseau **doivent** être en **partage** 
- pour utiliser les GPO, il faut que le dossier "PolicyDefinitions" soit dans le répertoire Policies: **SYSVOL>domain>policies**

## activer / désactiver le pare-feu windows

**chemin d'accés:**
=> configuration ordinateur => stratégie => modèles d'administration => réseau => connexions réseaux => Pare-feu windows
**paramètre:** "protéger toutes les connexions réseau"

## forcer la fermeture de session

**chemin d'accés:**
=> configuration ordinateur => stratégie => paramètres windows => paramètres de sécurité => options de sécurité
**paramètre:** "serveur réseau microsoft: déconnecter les clients à l'expiration du délai de la durée de session"
**paramètre:** "sécurité réseau: forcer la fermeture de session quand les horaires de connexion expirent"


## mappage d'un dossier sur un serveur distant

**chemin d'accés:** 
=> configuration utilisateur => préférences => paramètres windows => mappage de lecteurs
**paramètres:** 
*emplacement:* liens réseau vers le dossier **en partage** à mapper
*reconnecter* nom du lecteur mapper sur les postes clients

## accès à distance

**chemin d'accés:**
=> configuration d'ordinateur => réseau => connexions réseau => Pare-feu windows => profil du domaine
**paramètre:** autoriser l'exception de partage de fichiers entrants et d'imprimantes
indiquer l'adresse du réseau à autoriser
**paramètre:** protéger toutes les connexions réseau

**chemin d'accès:** => configuration ordinateur => stratégie => modèles d'administration => composants windows => services bureau à distance => hote de la session bureau a distance => connexions
**paramètre:** autoriser les utilisateurs à se connecter à distance à l'aide des services de bureau à distance
**paramètre:** définir les règles pour le contrôle à distance des sessions utilisateurs des services bureau à distance

**chemin d'accés:** => configuration ordinateur => préfèrences => paramètres du panneau de configuration => utilisateurs et groupe locaux 
**paramètre:** clic droit => nouveau => groupe local
*indiquer le/les groupe qui auront le droit d'utilisé le bureau à distance

## bloquer l'accès à des logiciels 

**chemin d'accés:** => configuration utilisateur => stratégie => modèle d'administration => système
**paramètre:** Ne pas exécuter les applications windows spécifiés
*indiqué les fichier en **.exe***

## modification du fond d'écran 

**chemin d'accès:** => configuration utilisateur => stratégie => modèle d'administration => bureau => bureau 
**paramètres:** papier peint du bureau
*préciser le chemin **réseau** ainsi que **l'extention** du fond d'écran*

## installation de logiciels

**chemin d'accès:** => configuration ordinateur => stratégie => paramètres logiciels 
**paramètres:** indiquer le chemin **réseau** vers l'application à installer.
*l'application doit être en **.msi***

## bloquer le panneau de configuration & l'ajout/suppression des programmes

**chemin d'accès:** => configuration utilisateur => stratégie => modèles d'administration => panneau de configuration 
**paramètres:** interdire l'accès au panneau de configuration et à l'application paramètres du PC

**chemin d'accès:** => configuration utilisateur => stratégie => modèles d'administration => panneau de configuration => ajouter ou supprimer des programmes
**paramètres:** supprimer l'application ajouter ou supprimer des programmes