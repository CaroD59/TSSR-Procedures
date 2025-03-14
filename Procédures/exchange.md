
# Installation d'Exchange

## Configuration PC AD DS, DNS, DHCP minimum -> VOIR WINDOWS SERVER 2016

### PC 1 : AD
Processeur: 1, RAM: 4096 Mo, Disque: 40 Go , Host-Only 

1. Install-WindowsFeature RSAT-ADDS
2. Redémarrer le serveur
3. Mettre l'ISO dans le C:/
4. Créer dans le C:/ -> C:/EXCH
5. Aller dans l'ISO et tout copier pour le coller dans le C:/EXCH
6. Ouvrir le CMD, aller dans C:/EXCH et taper les commandes suivantes
7. Setup.exe /PrepareSchema /IAcceptExchangeServerLicenseTerms
8. Setup.exe /PrepareAD /OrganizationName:M2I /IAcceptExchangeServerLicenseTerms
9. Setup.exe /PrepareAllDomains /IAcceptExchangeServerLicenseTerms 
10. Redémarrer le serveur

### PC 2 : Server
Processeur: 1, RAM: 8192 Mo, Disque: 120 Go , Host-Only 

1. Aller dans le PowerShell et taper ce script :
Liste des fonctionnalités nécessaires pour Exchange 2016
```powershell
$features = @(
    "NET-Framework-45-Features",
    "NET-WCF-HTTP-Activation45",
    "Web-Mgmt-Console",
    "WAS-Process-Model",
    "Web-Asp-Net45",
    "Web-Basic-Auth",
    "Web-Client-Auth",
    "Web-Digest-Auth",
    "Web-Dir-Browsing",
    "Web-Dyn-Compression",
    "Web-Http-Errors",
    "Web-Http-Logging",
    "Web-Http-Redirect",
    "Web-Http-Tracing",
    "Web-ISAPI-Ext",
    "Web-ISAPI-Filter",
    "Web-Lgcy-Mgmt-Console",
    "Web-Metabase",
    "Web-Mgmt-Service",
    "Web-Net-Ext45",
    "Web-Request-Monitor",
    "Web-Server",
    "Web-Stat-Compression",
    "Web-Static-Content",
    "Web-Windows-Auth",
    "Web-WMI",
    "Windows-Identity-Foundation",
    "RPC-over-HTTP-proxy",
    "RSAT-Clustering",
    "RSAT-Clustering-CmdInterface",
    "RSAT-Clustering-Mgmt",
    "RSAT-Clustering-Powershell"
)

foreach ($feature in $features) {
    $featureStatus = Get-WindowsFeature -Name $feature

    if ($featureStatus.Installed) {
        Write-Host "$feature est déjà installé." -ForegroundColor Green
    }
    else {
        Write-Host "Installation de $feature..." -ForegroundColor Yellow
        Install-WindowsFeature -Name $feature
        if ($?) {
            Write-Host "$feature a été installé avec succès." -ForegroundColor Green
        }
        else {
            Write-Host "Erreur lors de l'installation de $feature." -ForegroundColor Red
        }
    }
}

Write-Host "Vérification des fonctionnalités installées..." -ForegroundColor Cyan
Get-WindowsFeature | Where-Object {$_.Installed -eq $true} | Format-Table -Property Name, Installed
```

2. Redémarrer le serveur
3. Créer un dossier dans C:/ et mettre les utilitaires
4. Ouvrir le CMD, aller dans le dossier créé et taper les .exe
5. Mettre l'ISO dans la machine
6. Aller dans l'ISO et tout copier et tout coller dans un dossier dédié créé dans le C:/
7. Lancer le Setup, choisir de ne pas mettre à jour le serveur, et choisir boîte mail. Choix par défaut ensuite.

## Création nouveau compte Exchange

### Ajout des cmdlets Exchange à PowerShell
```powershell
Add-PSSnapin Microsoft.Exchange.Management.PowerShell.SnapIn
```
- Affiche les informations des serveurs Exchange
```powershell
Get-ExchangeServer | Format-Table Name, Edition, AdminDisplayVersion, Roles
```

- Création d'une nouvelle boîte aux lettres :
```powershell
New-Mailbox -Alias CDorchies -Name CDorchies -FirstName Caroline -LastName Dorchies -DisplayName "Caroline Dorchies" -UserPrincipalName cdorchies@afci.local -Password (ConvertTo-SecureString -String 'Azerty1' -AsPlainText -Force) -ResetPasswordOnNextLogon $true
```

### Vérification :
```powershell
Get-Mailbox -Identity "CDorchies"
Get-ADUser -Identity "CDorchies"
Get-Mailbox
```

## BAL en lot

1. Dans le C:/ :
   - Créer un dossier CSV
   - Dans le dossier CSV, créer un fichier txt qui ressemble à ça :
     Exemple nom du fichier : usersaimporter
     Exemple contenu du fichier
     ```txt
     Alias,Nom,UPN
     Djoe,Doe,djoe@afci.local
     Cclaire,Collart,cclaire@afci.local
     ```

2. Créer dans la machine AD la nouvelle organisation cible :
```powershell
New-ADOrganizationalUnit -Name "Salle4" -Path "DC=afci,DC=local"
```

3. Créer dans la machine Server, aller sur Exchange Management Shell et dans le dossier CSV :
```powershell
Import-Csv usersaimporter.txt | ForEach-Object {
    New-Mailbox -Alias $_.Alias -Name $_.Nom -UserPrincipalName $_.UPN -OrganizationalUnit "OU=Salle4,OU=Utilisateurs,DC=afci,DC=local" -Password $Password
}
```

4. Vérifier les OU :
```powershell
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName
```

## Destinaires

### DISTRIBUTION

#### Créer une liste de distribution
- Dans le centre d'administration Exchange, aller dans groupes et le petit + : "Créer un groupe de distribution"
- Remplir les informations et ajouter les membres
- Ajouter les propriétaires du groupe en tant que membres coché
- Tout sur fermé
#### Créer un groupe de distribution dans Exchange Management Shell :
```powershell
New-DistributionGroup -Name "charifgroupe" -Alias "charifgroupe" -PrimarySmtpAddress "charifgroupe@afci.local" -Type Distribution
Get-DistributionGroup -Identity "charifgroupe"
```

#### Créer une liste de distribution dynamique
- Dans le centre d'administration Exchange, aller dans groupes et le petit + : "Créer un groupe de distribution dynamique"
- Remplir les informations et ajouter les membres
- Ajout des règles pour critères d'appartenance à ce groupe. Si la valeur pour l'attribut sélectionné correspond à cette valeur que vous avez définie, le destinataire reçoit un message envoyé à ce groupe.

#### Afficher les membres d’un groupe de distribution dynamique :
```powershell
$a = Get-DynamicDistributionGroup "GROUPE DYN 1"
Get-Recipient -RecipientPreviewFilter $a.RecipientFilter -OrganizationalUnit $a.RecipientContainer
```

## Contact

#### Créer un contact de messagerie
- Aller dans contacts et remplir les informations (tout par défaut)
- Dans le Exchange Management Shell :
```powershell
New-MailContact -Name "Anne Campos" -ExternalEmailAddress "acampos@m2i.com" -OrganizationalUnit "Users"
Get-MailUser
```

#### Créer un utilisateur de messagerie
- Aller dans contacts et remplir les informations (tout par défaut)
- Dans le Exchange Management Shell :
```powershell
New-MailUser -Name "Emma Meny" -Alias emeny -ExternalEmailAddress "emeny@m2i.com" -FirstName "Emma" -LastName "Meny" -UserPrincipalName "emeny@afci.local" -Password (ConvertTo-SecureString -String "Azerty1" -AsPlainText -Force)
Get-MailUser
```

## Base de données

- Aller dans serveurs, bases de données, créer une nouvelle base données et remplir les informations
- Dans le Exchange Management Shell :
1. Créer un dossier "MaBDD" dans le C:/ (peu importe le chemin)
2. Créer la base de données
```powershell
New-MailboxDatabase -Name "MaBase2" -Server EXCH-SERVER -EdbFilePath C:\MaBase2\Mabase2.edb -LogFolderPath C:\Mabase2\LogBase2
```
3. Mise en marche de la base de données
```powershell
Mount-Database -Identity MaBase2
```
4. Déplacer une base de données
```powershell
Move-DatabasePath -Identity MaBase2 -EdbFilePath "c:\MaBase3\MaBase3.edb" -LogFolderPath C:\MaBase3\LogFolder
```

### Vérification :
```powershell
Get-MailboxDatabase
Get-MailboxDatabase | Format-Table Name, Mounted, EdbFilePath, LogFolderPath
Get-MailboxDatabase | Select Name, DatabaseSize, AvailableNewMailboxSpace
```

## Boîte à lettres partagée

1. Aller dans destinataires > boîte aux lettres partagée et le petit + : "Nouvelle boîte aux lettres partagée"
2. Remplir les informations
3. Aller sur "Délégation de boîte aux lettres" et rajouter les utilisateurs ayant les accès à cette boîte aux lettres
4. Aller tout en haut à droite, aller sur son compte, ouvrir une autre boîte aux lettres, et rajouter le mail partagé
- ou
5. Clique droit sur notre nom à gauche; et cliquer sur "Ajouter dossier partagé" et rajouter le mail partagé

## Boîte à lettres de ressources

- Aller dans destinataires > ressources et le petit + : "Nouvelle boîte aux lettres de salle
- Remplir les informations et les critères dans options de réservation
- Dans le Exchange Management Shell :
1. Créer une boîte aux lettres de salle ou équipement
```powershell
New-Mailbox -Name "Réunion 1" -DisplayName "Salle de réunion 1" -Room
New-Mailbox -Alias "Reunion2" -Name "Salle de réunion 2" -DisplayName "Salle de réunion 2" -UserPrincipalName "reunion2@afci.local" –room
```

2. Pour modifier son nom affiché
```powershell
Set-Mailbox "Ancien nom" -DisplayName "Nouveau nom"
```

3. Vérification
```powershell
Get-CalendarProcessing "Reunion1" | fl
```

## Règles

### Afficher une règle du flux de courriers
```powershell
Get-TransportRule
```

#### 1. Filtre par pièce jointe
- Aller dans flux de messagerie, aller dans le petit plus > Filtrer les messages par taille
- Toute pièce jointe... > L'extension du fichier comprend ces mots... > Bloquer le message avec une explication
- Remplir les informations
- Dans le Exchange Management Shell :
```powershell
New-TransportRule -Name "Bloquer toutes les pièces jointes sauf PDF" `
-AttachmentExtensionMatchesWords "exe", "bat", "zip", "rar", "docx", "xlsx", "pptx", "txt" `
-RejectMessageEnhancedStatusCode "5.7.1" `
-RejectMessageReasonText "Seuls les fichiers PDF sont autorisés en pièce jointe."
```

#### 2. Filtre par taille
- Aller dans flux de messagerie, aller dans le petit plus > Filtrer les messages par taille
- La taille du message et supérieur ou égal à... > Bloquer le message avec une explication
- Remplir les informations
- Dans le Exchange Management Shell :
```powershell
New-TransportRule -Name "Bloquer l'envoi d'un fichier qui dépasse 1 Mo" `
-AttachmentSizeOver 1MB `
-RejectMessageEnhancedStatusCode "5.7.1" `
-RejectMessageReasonText "Vous ne pouvez pas envoyer un fichier supérieur à 1 Mo."
```

#### 3. Filtre pour bloquer l'envoi à un destinataire
- Aller dans flux de messagerie, aller dans le petit plus > Filtrer les messages par taille
- Le destinaire est... > ... cette personne > Bloquer le message avec une explication
- Remplir les informations
- Dans le Exchange Management Shell :
```powershell
New-TransportRule -Name "Bloquer l'envoi vers un destinataire" `
-RecipientAddressContains "sara@yasser59.onmicrosoft.com" `
-RejectMessageEnhancedStatusCode "5.7.1" `
-RejectMessageReasonText "Vous ne pouvez pas envoyer un message à ce destinataire."
```

### Désactiver une règle
- Dans le Exchange Management Shell :
```powershell
Disable-TransportRule -Identity "Bloquer l'envoi vers un destinataire"
```

### Réactiver une règle
- Dans le Exchange Management Shell :
```powershell
Enable-TransportRule -Identity "Bloquer l'envoi vers un destinataire"
```

### Vérification
```powershell
Get-TransportRule | Format-Table Name,State,Priority
```

## GESTION DES ERREURS

### Sur interface
- Si problème scripting : https://support.microsoft.com/fr-fr/topic/proc%C3%A9dure-d-activation-de-javascript-dans-windows-88d27b37-6484-7fc0-17df-872f65168279
- Si problème cookie : https://support.microsoft.com/fr-fr/office/activer-les-cookies-6b018d22-1d24-43d9-8543-3d35ddb2cb52
