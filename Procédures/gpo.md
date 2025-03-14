# GPO

### Informations générales :
- Les dossiers accessibles sur le réseau **doivent** être en **partage**.
- Pour utiliser les GPO, il faut que le dossier "PolicyDefinitions" soit dans le répertoire Policies : **SYSVOL > domain > policies**.

## Activer / désactiver le pare-feu Windows

**Chemin d'accès :**  
`Configuration ordinateur > Stratégie > Modèles d'administration > Réseau > Connexions réseau > Pare-feu Windows`  
**Paramètre :**  
- "Protéger toutes les connexions réseau"

## Forcer la fermeture de session

**Chemin d'accès :**  
`Configuration ordinateur > Stratégie > Paramètres Windows > Paramètres de sécurité > Options de sécurité`  
**Paramètres :**  
- "Serveur réseau Microsoft : déconnecter les clients à l'expiration du délai de la durée de session"  
- "Sécurité réseau : forcer la fermeture de session quand les horaires de connexion expirent"

## Mappage d'un dossier sur un serveur distant

**Chemin d'accès :**  
`Configuration utilisateur > Préférences > Paramètres Windows > Mappage de lecteurs`  
**Paramètres :**  
- **Emplacement :** Liens réseau vers le dossier **en partage** à mapper  
- **Reconnecter :** Nom du lecteur mappé sur les postes clients

## Accès à distance

**Chemin d'accès :**  
`Configuration ordinateur > Réseau > Connexions réseau > Pare-feu Windows > Profil du domaine`  
**Paramètre :**  
- Autoriser l'exception de partage de fichiers entrants et d'imprimantes  
- Indiquer l'adresse du réseau à autoriser  
- "Protéger toutes les connexions réseau"

**Chemin d'accès :**  
`Configuration ordinateur > Stratégie > Modèles d'administration > Composants Windows > Services Bureau à distance > Hôte de la session Bureau à distance > Connexions`  
**Paramètre :**  
- Autoriser les utilisateurs à se connecter à distance à l'aide des services de bureau à distance  
- Définir les règles pour le contrôle à distance des sessions utilisateurs des services Bureau à distance

**Chemin d'accès :**  
`Configuration ordinateur > Préférences > Paramètres du panneau de configuration > Utilisateurs et groupes locaux`  
**Paramètre :**  
- Clic droit => Nouveau => Groupe local  
- Indiquer le/les groupe(s) qui auront le droit d'utiliser le bureau à distance

## Bloquer l'accès à des logiciels

**Chemin d'accès :**  
`Configuration utilisateur > Stratégie > Modèles d'administration > Système`  
**Paramètre :**  
- "Ne pas exécuter les applications Windows spécifiées"  
- Indiquer les fichiers en **.exe**

## Modification du fond d'écran

**Chemin d'accès :**  
`Configuration utilisateur > Stratégie > Modèles d'administration > Bureau > Bureau`  
**Paramètres :**  
- "Papier peint du bureau"  
- Préciser le chemin **réseau** ainsi que **l'extension** du fond d'écran

## Installation de logiciels

**Chemin d'accès :**  
`Configuration ordinateur > Stratégie > Paramètres logiciels`  
**Paramètres :**  
- Indiquer le chemin **réseau** vers l'application à installer.  
- *L'application doit être en **.msi***

## Bloquer le panneau de configuration & l'ajout/suppression des programmes

**Chemin d'accès :**  
`Configuration utilisateur > Stratégie > Modèles d'administration > Panneau de configuration`  
**Paramètres :**  
- "Interdire l'accès au panneau de configuration et à l'application Paramètres du PC"

**Chemin d'accès :**  
`Configuration utilisateur > Stratégie > Modèles d'administration > Panneau de configuration > Ajouter ou supprimer des programmes`  
**Paramètres :**  
- "Supprimer l'application Ajouter ou supprimer des programmes"
