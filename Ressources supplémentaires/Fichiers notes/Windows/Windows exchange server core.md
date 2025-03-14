## Mise en place d'un serveur windows exchange en mode core



#### commande permettant à PowerShell de connaitre les commandes lié à Exchange
> Add-PSSnapin Microsoft.Exchange.Management.PowerShell.SnapIn
 
 ------------------------------------------------------------------
### création d'une boite mail quand l'utilisateur n'existe pas
		$password = Read-Host "entre le mot de passe" -AsSecureString Import-Csv "C:\mail_a_cree.csv" | foreach {New-MailBox -Alias $_.Alias -name $_.Nom -UserPrincipalName $_.UPN -OrganizationalUnit $_.UNITOU -password $password}

Notes:
>- tout ce qui es "$_._mot_" correspond au nom de vos colonne à faire coïcidé avec les paramètres
>- "mail_a_cree.csv" est le nom de votre fichier contenant les utilisateurs au format csv
>- les unité d'organisation doivent être crée **AVANT** d'exécuter la commande

------------------------------------------------------------------
### création d'un groupe de distribution dynamique
> Enable-DistributionGroup -Identity "charifgroupe"

**ATTENTION** le groupe doit être crée **AVANT** d'exécuter la commande

------------------------------------------------------------------
### afficher les utilisateurs présents dans le groupe 
	$a = Get-DynamicDistributionGroup "GROUPE DYN 1" Get-Recipient -RecipientPreviewFilter $a.RecipientFilter -OrganizationalUnit $a.RecipientContainer

------------------------------------------------------------------
### crée un utilisateurs externe sans accés aux machine du domaine
	New-MailUser -Name "Adnre INESTA" -Alias "a.inesta" -ExternalEmailAddress "a.inesta@gmail.com" -FirstName Andres -LastName INESTA -UserPrincipalName a.inesta@M2i.local -Password (ConvertTo-SecureString -String "Azerty1" -AsPlainText -Force)
	
------------------------------------------------------------------
### création d'une base de donnée exchange
	New-MailboxDatabase -name "MaBase2" -Server exchange -EdbFilePath c:\Mabase2\Mabase2.edb -LogFolderPath c:\Mabase2\LogBase2

Notes:
>- le paramètre "EdbFilePath" : demande le chemin vers le fichier qui contiendra les données de la base de donnée
>- la paramètre "LogFolderPath" : demande le chemin vers le fichier de log
>- le paramètre "Server" : demande le serveur sur lequel la base de donnée doit être stocké (peut etre un autre serveur que celui sur lequel ce trouve exchange)
	
les fichiers **ne doivent pas** être crée au préalable

------------------------------------------------------------------
### mise en route de la base de donnée crée
	 Mount-Database -Identity MaBase2

Notes:
>- par défaut les bases de données crée en ligne de commandes ne sont pas "monté" (elles ne sont pas mise en route)	

------------------------------------------------------------------
### déplacer une base de donnée existante
	Move-DatabasePath -Identity Mabase2 -EdbFilePath "c:\MaBase3\MaBase3.edb" -LogFolderPath C:\MaBase3\LogFolder

Notes:
>- déplace la base de donnée nommé "Mabase2" vers le chemin donnée à "EdbFilePath" pour le fichier de la base de donnée et vers "LogFolderPath" pour le fichier de log
>- une confirmation est demandé pour le déplacement et si la base de donnée est monté, une confirmation sera demandé pour la démonté temporairemment (une fois le déplacement fini elle sera remonté automatiquement)

------------------------------------------------------------------


## Boîte à lettres de ressources

 Aller dans destinataires > ressources et le petit + : "Nouvelle boîte aux lettres de salle"
 > Remplir les informations et les critères dans options de réservation

------------------------------------------------------------------
### Dans le Exchange Management Shell : 
*Pour créer une boîte aux lettres de salle*
 
	New-Mailbox -Name "Réunion 1" -DisplayName "Salle de réunion 1" -Room 

Exemple : 

	New-Mailbox -Alias "Reunion2" -Name "Salle de réunion 2" -DisplayName "Salle de réunion 2" -UserPrincipalName "reunion2@afci.local" –room
 
 ------------------------------------------------------------------
### Pour créer une boîte aux lettres d'équipement
	New-Mailbox -Name "Réunion 1" -DisplayName "Salle de réunion 1" -Equipment

------------------------------------------------------------------
### Pour modifier son nom affiché
	Set-Mailbox "Ancien nom" -DisplayName "Nouveau nom"

------------------------------------------------------------------
#### Vérification
	Get-CalendarProcessing "Reunion1" | fl


------------------------------------------------------------------
# Règles

### Afficher une règle du flux de courriers
	Get-TransportRule

## Filtre par pièce jointe
 Aller dans flux de messagerie, aller dans le petit plus > Filtrer les messages par taille
 Toute pièce jointe > L'extension du fichier comprend ces mots > Bloquer le message avec une explication

 __Remplir les informations__
 Dans le Exchange Management Shell :

	New-TransportRule -Name "Bloquer toutes les pièces jointes sauf PDF" ` -AttachmentExtensionMatchesWords "exe", "bat", "zip", "rar", "docx", "xlsx", "pptx", "txt" ` -RejectMessageEnhancedStatusCode "5.7.1" ` -RejectMessageReasonText "Seuls les fichiers PDF sont autorisés en pièce jointe."

------------------------------------------------------------------
## Filtre par taille
Aller dans flux de messagerie, aller dans le petit plus > Filtrer les messages par taille
La taille du message et supérieur ou égal à... > Bloquer le message avec une explication
### Remplir les informations
Dans le Exchange Management Shell :

	New-TransportRule -Name "Bloquer l'envoi d'un fichier qui dépasse 1 Mo" ` -AttachmentSizeOver 1MB ` -RejectMessageEnhancedStatusCode "5.7.1" ` -RejectMessageReasonText "Vous ne pouvez pas envoyer un fichier supérieur à 1 Mo."

------------------------------------------------------------------
### Filtre pour bloquer l'envoi à un destinataire
Aller dans flux de messagerie, aller dans le petit plus > Filtrer les messages par taille
Le destinaire est >  cette personne > Bloquer le message avec une explication

### Remplir les informations
Dans le Exchange Management Shell :

	New-TransportRule -Name "Bloquer l'envoi vers un destinataire" ` -RecipientAddressContains "sara@yasser59.onmicrosoft.com" ` -RejectMessageEnhancedStatusCode "5.7.1" ` -RejectMessageReasonText "Vous ne pouvez pas envoyer un message à ce destinataire."

## Désactiver une règle

Dans le Exchange Management Shell :

	Disable-TransportRule -Identity "Bloquer l'envoi vers un destinataire"

## Réactiver une règle

Dans le Exchange Management Shell :

	Enable-TransportRule -Identity "Bloquer l'envoi vers un destinataire"

## Vérification

	Get-TransportRule | Format-Table Name,State,Priority