# Mise en place d'un serveur Windows Exchange en mode Core

## Commande permettant à PowerShell de connaître les commandes liées à Exchange
```powershell
Add-PSSnapin Microsoft.Exchange.Management.PowerShell.SnapIn
```

---

## Création d'une boîte mail quand l'utilisateur n'existe pas
```powershell
$password = Read-Host "Entrez le mot de passe" -AsSecureString
Import-Csv "C:\mail_a_cree.csv" | foreach {
    New-MailBox -Alias $_.Alias -Name $_.Nom -UserPrincipalName $_.UPN -OrganizationalUnit $_.UNITOU -Password $password
}
```
**Notes :**
- Tout ce qui est `$_._mot_` correspond au nom de vos colonnes à faire coïncider avec les paramètres.
- `mail_a_cree.csv` est le nom de votre fichier contenant les utilisateurs au format CSV.
- Les unités d'organisation doivent être créées **avant** d'exécuter la commande.

---

## Création d'un groupe de distribution dynamique
```powershell
Enable-DistributionGroup -Identity "charifgroupe"
```
**ATTENTION :** Le groupe doit être créé **avant** d'exécuter la commande.

---

## Afficher les utilisateurs présents dans le groupe
```powershell
$a = Get-DynamicDistributionGroup "GROUPE DYN 1"
Get-Recipient -RecipientPreviewFilter $a.RecipientFilter -OrganizationalUnit $a.RecipientContainer
```

---

## Créer un utilisateur externe sans accès aux machines du domaine
```powershell
New-MailUser -Name "Andres INESTA" -Alias "a.inesta" -ExternalEmailAddress "a.inesta@gmail.com" -FirstName Andres -LastName INESTA -UserPrincipalName a.inesta@M2i.local -Password (ConvertTo-SecureString -String "Azerty1" -AsPlainText -Force)
```

---

## Création d'une base de données Exchange
```powershell
New-MailboxDatabase -Name "MaBase2" -Server exchange -EdbFilePath C:\Mabase2\Mabase2.edb -LogFolderPath C:\Mabase2\LogBase2
```
**Notes :**
- Le paramètre `EdbFilePath` demande le chemin vers le fichier qui contiendra les données de la base de données.
- Le paramètre `LogFolderPath` demande le chemin vers le fichier de log.
- Le paramètre `Server` demande le serveur sur lequel la base de données doit être stockée (peut être un autre serveur que celui sur lequel se trouve Exchange).
- Les fichiers **ne doivent pas** être créés au préalable.

---

## Mise en route de la base de données créée
```powershell
Mount-Database -Identity MaBase2
```
**Notes :**
- Par défaut, les bases de données créées en ligne de commande ne sont pas "montées" (elles ne sont pas mises en route).

---

## Déplacer une base de données existante
```powershell
Move-DatabasePath -Identity Mabase2 -EdbFilePath "C:\MaBase3\MaBase3.edb" -LogFolderPath C:\MaBase3\LogFolder
```
**Notes :**
- Déplace la base de données nommée `Mabase2` vers le chemin donné à `EdbFilePath` pour le fichier de la base de données et vers `LogFolderPath` pour le fichier de log.
- Une confirmation est demandée pour le déplacement et si la base de données est montée, une confirmation sera demandée pour la démonter temporairement (une fois le déplacement fini, elle sera remontée automatiquement).

---

# Boîte aux lettres de ressources
## Création d'une boîte aux lettres de salle
Aller dans **Destinataires > Ressources** et cliquer sur **Nouvelle boîte aux lettres de salle**.
Remplir les informations et les critères dans **Options de réservation**.

### Dans l'Exchange Management Shell :
```powershell
New-Mailbox -Name "Réunion 1" -DisplayName "Salle de réunion 1" -Room
```
**Exemple :**
```powershell
New-Mailbox -Alias "Reunion2" -Name "Salle de réunion 2" -DisplayName "Salle de réunion 2" -UserPrincipalName "reunion2@afci.local" –Room
```

---

## Création d'une boîte aux lettres d'équipement
```powershell
New-Mailbox -Name "Réunion 1" -DisplayName "Salle de réunion 1" -Equipment
```

---

## Modifier le nom affiché d'une boîte aux lettres
```powershell
Set-Mailbox "Ancien nom" -DisplayName "Nouveau nom"
```

---

## Vérification
```powershell
Get-CalendarProcessing "Reunion1" | fl
```

---

# Règles
## Afficher une règle du flux de courriers
```powershell
Get-TransportRule
```

---

## Filtrer par pièce jointe
Aller dans **Flux de messagerie**, puis cliquer sur **+ > Filtrer les messages par taille**.
- **Toute pièce jointe** > **L'extension du fichier comprend ces mots** > **Bloquer le message avec une explication**.

### Dans l'Exchange Management Shell :
```powershell
New-TransportRule -Name "Bloquer toutes les pièces jointes sauf PDF" `
-AttachmentExtensionMatchesWords "exe", "bat", "zip", "rar", "docx", "xlsx", "pptx", "txt" `
-RejectMessageEnhancedStatusCode "5.7.1" `
-RejectMessageReasonText "Seuls les fichiers PDF sont autorisés en pièce jointe."
```

---

## Filtrer par taille
Aller dans **Flux de messagerie**, puis cliquer sur **+ > Filtrer les messages par taille**.
- **La taille du message est supérieure ou égale à...** > **Bloquer le message avec une explication**.

### Dans l'Exchange Management Shell :
```powershell
New-TransportRule -Name "Bloquer l'envoi d'un fichier qui dépasse 1 Mo" `
-AttachmentSizeOver 1MB `
-RejectMessageEnhancedStatusCode "5.7.1" `
-RejectMessageReasonText "Vous ne pouvez pas envoyer un fichier supérieur à 1 Mo."
```

---

## Filtrer pour bloquer l'envoi à un destinataire
Aller dans **Flux de messagerie**, puis cliquer sur **+ > Filtrer les messages par taille**.
- **Le destinataire est** > **Cette personne** > **Bloquer le message avec une explication**.

### Dans l'Exchange Management Shell :
```powershell
New-TransportRule -Name "Bloquer l'envoi vers un destinataire" `
-RecipientAddressContains "sara@yasser59.onmicrosoft.com" `
-RejectMessageEnhancedStatusCode "5.7.1" `
-RejectMessageReasonText "Vous ne pouvez pas envoyer un message à ce destinataire."
```

---

## Désactiver une règle
```powershell
Disable-TransportRule -Identity "Bloquer l'envoi vers un destinataire"
```

## Réactiver une règle
```powershell
Enable-TransportRule -Identity "Bloquer l'envoi vers un destinataire"
```

## Vérification des règles
```powershell
Get-TransportRule | Format-Table Name,State,Priority
```

