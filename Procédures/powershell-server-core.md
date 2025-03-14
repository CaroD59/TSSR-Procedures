# PowrShell- Installation Server Core

### Poste de travail
```powershell
- **IP** : 192.168.10.1
- **Domaine** : M2i-core.local
- **Forêt** : M2i-core.local
- **Ordinateur** : AD-M2I-1
- **Hostname** : AD-M2I-1.M2i-core.local
```

### Serveur
```powershell
- **IP** : 192.168.10.2
- **Domaine** : M2i-core.local
- **Forêt** : M2i-core.local
- **Ordinateur** : AD-M2I-1-Server
- **Hostname** : AD-M2I-1-Server.M2i-core.local
```

### TP
```powershell
- **IP** : 192.168.10.1
- **Domaine** : M2I.local
```

---

## PowerShell

```powershell
powershell
```

### Installer VMTools
```powershell
Get-PSDrive
cd d:
ls
. \setup.exe
```


### Renommer l'ordinateur
```powershell
Rename-Computer -NewName AD-M2I-1
Restart-Computer -Force
$env:COMPUTERNAME
```

### Désactiver le pare-feu
```powershell
netsh advfirewall set allprofiles state off
```

### Ajouter les services ADDS
```powershell
Add-WindowsFeature AD-Domain-Services
```

### Ajouter un domaine
```powershell
Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath "C:\Windows\ntds" -DomainMode "Win2012" -DomainName "M2I.local" -DomainNetbiosName "m2i-core" -ForestMode "Win2012" -InstallDns:$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:$false -SysvolPath "C:\Windows\SYSVOL" -Force:$true
```

### Changer l'IP
```powershell
netsh interface ipv4 set address name= Ethernet0 static 192.168.10.1 255.255.255.0
```

### Ajouter le DHCP
```powershell
Install-WindowsFeature -name dhcp -includemanagementtools
Add-DhcpServerInDC -DnsName M2I 192.168.10.1
Add-DhcpServerv4Scope -Name "M2I-Core" -StartRange 192.168.10.10 -EndRange 192.168.10.50 -SubnetMask 255.255.255.0
Set-DhcpServerv4OptionDefinition -OptionId 3 -DefaultValue 192.168.10.254
Set-DhcpServerv4OptionDefinition -OptionId 6 -DefaultValue 192.168.10.1
Set-DhcpServerv4OptionDefinition -OptionId 15 -DefaultValue M2I.local
```

### Vérifier l'installation
```powershell
Get-WindowsFeature -Name DHCP
Get-WindowsFeature -Name AD-Domain-Services
```

### Vérifier les informations
```powershell
Get-ADDomainController
Get-ComputerInfo | Select-Object CsDomain, CsDomainRole
```

### Vérifier l'étendue DHCP
```powershell
Get-DhcpServerv4Scope
```

---

## Serveur

### PowerShell

#### Renommer l'ordinateur
```powershell
Rename-Computer -NewName AD-M2I-1-Server
Restart-Computer (-force)
$env:COMPUTERNAME
```

#### Désactiver le pare-feu
```powershell
netsh advfirewall set allprofiles state off
```

#### Changer l'IP
```powershell
netsh interface ipv4 set address name= Ethernet0 static 192.168.10.2 255.255.255.0
```

#### Joindre la machine au domaine
- Pour connaître l'index de l'interface, faire : 
```powershell
Get-Netadapter
Set-DnsClientServerAddress -InterfaceIndex 9 -ServerAddresses 192.168.10.1
Add-Computer -ComputerName AD-M2I-1-Server -DomainName M2I-Core.local
```

/!\ Si ça ne fonctionne pas :
- Vérifier si on ping : `ping 192.168.10.1`
- Vérifier DNS : `nslookup M2I-Core.local`
- Vérifier le réseau : les adresses IP doivent être sur le même réseau !
- Redémarrer l'ordinateur

#### Rendre le serveur additionnel
```powershell
Add-WindowsFeature AD-Domain-Services
Restart-Computer (-force)
```

#### Faire échapper deux fois quand on demande le mot de passe, sélectionner "autre utilisateur" et remplir les informations.

```powershell
$password = ConvertTo-SecureString -AsPlainText -String Azerty1 -force
Install-ADDSDomainController -DomainName M2I-Core.local -DatabasePath "C:\Windows\NTDS" -LogPath "C:\Windows\NTDS" -SysvolPath "C:\Windows\SYSVOL" -InstallDns -ReplicationSourceDC AD-M2I-1.M2i-core.local -SafeModeAdministratorPassword $password -NoRebootOnCompletion
Restart-Computer (-force)
```


# EXERCICES

### Créer l'OU principale M2I
```powershell
New-ADOrganizationalUnit -Name "M2I" -Path "DC=M2I,DC=local"
```

### Créer les sous-OUs sous M2I
```powershell
New-ADOrganizationalUnit -Name "Stagiaire" -Path "OU=M2I,DC=M2I,DC=local"
New-ADOrganizationalUnit -Name "Formateur" -Path "OU=M2I,DC=M2I,DC=local"
New-ADOrganizationalUnit -Name "Administration" -Path "OU=M2I,DC=M2I,DC=local"
```

### Créer les sous-OUs sous Stagiaire
```powershell
New-ADOrganizationalUnit -Name "TSSR" -Path "OU=Stagiaire,OU=M2I,DC=M2I,DC=local"
New-ADOrganizationalUnit -Name "Web-Design" -Path "OU=Stagiaire,OU=M2I,DC=M2I,DC=local"


Get-ADOrganizationalUnit -Filter * | Select Name, DistinguishedName
```

7. 
### Définir un mot de passe sécurisé pour les comptes
```powershell
$Password = ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force
```

### Créer user1 dans l'OU Web-Design
```powershell
New-ADUser -Name "user1" `
    -GivenName "User" `
    -Surname "User1" `
    -SamAccountName "user1" `
    -UserPrincipalName "user1@m2i.local" `
    -Path "OU=Web-Design,OU=Stagiaire,OU=M2I,DC=M2I,DC=local" `
    -AccountPassword $Password `
    -Enabled $true
```

### Créer user2 dans l'OU Web-Design
```powershell
New-ADUser -Name "user2" `
    -GivenName "User" `
    -Surname "User2" `
    -SamAccountName "user2" `
    -UserPrincipalName "user2@m2i.local" `
    -Path "OU=Web-Design,OU=Stagiaire,OU=M2I,DC=M2I,DC=local" `
    -AccountPassword $Password `
    -Enabled $true
```

### Créer user3 dans l'OU Web-Design
```powershell
New-ADUser -Name "user3" `
    -GivenName "User" `
    -Surname "User3" `
    -SamAccountName "user3" `
    -UserPrincipalName "user3@m2i.local" `
    -Path "OU=Web-Design,OU=Stagiaire,OU=M2I,DC=M2I,DC=local" `
    -AccountPassword $Password `
    -Enabled $true


Get-ADUser -Filter * -SearchBase "OU=Web-Design,OU=Stagiaire,OU=M2I,DC=M2I,DC=local" | Select Name, SamAccountName
```

8. 
```powershell
New-ADGroup -Name "G1" `
    -DisplayName "G1" `
    -GroupCategory Security `
    -GroupScope Global `
    -Path "OU=Formateur,OU=M2I,DC=M2I,DC=local"
```

9. 
```powershell
New-ADGroup -Name "G2" `
    -DisplayName "G2" `
    -GroupCategory Distribution `
    -GroupScope Universal `
    -Path "OU=Administration,OU=M2I,DC=M2I,DC=local"
```

10. 
```powershell
Remove-ADUser -Identity "user1" -Confirm:$false
Get-ADUser -Identity "user1"
```

11.
```powershell
Disable-ADAccount -Identity "user2"
Get-ADUser -Identity "user2" | Select-Object SamAccountName, Enabled
```

12. 
```powershell
Add-ADGroupMember -Identity "G1" -Members "user3"
Get-ADGroupMember -Identity "G1"
```

13. 
```powershell
Add-ADGroupMember -Identity "G2" -Members "user2"
Get-ADGroupMember -Identity "G2"
```

14. 
```powershell
Enable-ADAccount -Identity "user2"
Get-ADUser -Identity "user2" | Select-Object SamAccountName, Enabled
```

15. 
```powershell
$users = @(
    [PSCustomObject]@{Nom='Dupont'; Prénom='Jean'; Nom_SAM='j.dupont'; UPN='j.dupont@M2I.local'},
    [PSCustomObject]@{Nom='Martin'; Prénom='Claire'; Nom_SAM='c.martin'; UPN='c.martin@M2I.local'}
)
$filePath = "C:\Users\Administrateur\Stagiaires_TSSR.csv"
$users | Export-Csv -Path $filePath -NoTypeInformation -Delimiter ';'
Test-Path $filePath
```

### Demander à l'utilisateur de saisir un mot de passe sécurisé
```powershell
$mdp = Read-Host "Veuillez saisir le mot de passe" -AsSecureString
```

### Importer les utilisateurs depuis le fichier CSV et créer les comptes dans Active Directory
```powershell
Import-Csv C:\Users\Administrateur\Stagiaires_TSSR.csv -Delimiter ';' | ForEach-Object {
    New-ADUser -SamAccountName $_.Nom_SAM `
               -UserPrincipalName $_.UPN `
               -GivenName $_.Prénom `
               -Surname $_.Nom `
               -AccountPassword $mdp `
               -CannotChangePassword $false `
               -ChangePasswordAtLogon $true `
               -PasswordNeverExpires $false `
               -Enabled $true `
               -Path "OU=TSSR,OU=Stagiaire,OU=M2I,DC=M2I,DC=local"
}


cd C:\Users\Administrateur

.\Import_Stagiaires_TSSR.ps1


Get-ADUser -Filter * -SearchBase "OU=TSSR,OU=Stagiaire,OU=M2I,DC=M2I,DC=local"
```

16. 
```powershell
Remove-ADOrganizationalUnit -Identity "OU=Web-Design,OU=Stagiaire,OU=M2I,DC=M2I,DC=local" -Recursive
Get-ADOrganizationalUnit -Filter {Name -eq "Web-Design"} -SearchBase "OU=Stagiaire,OU=M2I,DC=M2I,DC=local"
```

17. 
### Affiche un menu pour choisir l'opération à effectuer
```powershell
function Show-Menu {
    Clear-Host
    Write-Host "===================================" -ForegroundColor Cyan
    Write-Host "   Choisissez une opération :      "
    Write-Host "===================================" -ForegroundColor Cyan
    Write-Host "1. Ajouter un utilisateur"
    Write-Host "2. Ajouter un groupe"
    Write-Host "3. Ajouter une Unité Organisationnelle (OU)"
    Write-Host "4. Quitter"
}
```

### Fonction pour ajouter un utilisateur
```powershell
function Add-User {
    $Nom = Read-Host "Entrez le nom de l'utilisateur"
    $Prenom = Read-Host "Entrez le prénom de l'utilisateur"
    $Nom_SAM = Read-Host "Entrez le Nom SAM (ex: j.dupont)"
    $UPN = Read-Host "Entrez l'UPN (ex: j.dupont@M2I.local)"
    $Password = Read-Host "Entrez le mot de passe de l'utilisateur" -AsSecureString
    
    # Crée l'utilisateur dans Active Directory
    New-ADUser -SamAccountName $Nom_SAM `
               -UserPrincipalName $UPN `
               -Name "$Prenom $Nom" `
               -GivenName $Prenom `
               -Surname $Nom `
               -AccountPassword $Password `
               -CannotChangePassword $false `
               -ChangePasswordAtLogon $true `
               -PasswordNeverExpires $false `
               -Enabled $true

    Write-Host "L'utilisateur $Nom_SAM a été ajouté avec succès !" -ForegroundColor Green
}
```

### Fonction pour ajouter un groupe
```powershell
function Add-Group {
    $NomGroupe = Read-Host "Entrez le nom du groupe"
    $Scope = Read-Host "Entrez l'étendue du groupe (ex: Global, DomainLocal, Universal)"
    $Category = Read-Host "Entrez le type de groupe (ex: Security, Distribution)"
    
    # Crée le groupe dans Active Directory
    New-ADGroup -Name $NomGroupe `
                -GroupScope $Scope `
                -GroupCategory $Category `
                -DisplayName $NomGroupe `
                -SamAccountName $NomGroupe `
                -Enabled $true

    Write-Host "Le groupe $NomGroupe a été créé avec succès !" -ForegroundColor Green
}
```

### Fonction pour ajouter une Unité Organisationnelle (OU)
```powershell
function Add-OU {
    $NomOU = Read-Host "Entrez le nom de l'Unité Organisationnelle"
    $ParentOU = Read-Host "Entrez le chemin de l'OU parent (ex: OU=M2I,DC=M2I,DC=local)"
    
    # Crée l'OU dans Active Directory
    New-ADOrganizationalUnit -Name $NomOU -Path "DC=M2I,DC=local\$ParentOU"

    Write-Host "L'OU $NomOU a été créée avec succès !" -ForegroundColor Green
}
```

# Fonction principale pour exécuter le menu et les options
```powershell
function Main {
    do {
        Show-Menu
        
        # Demande à l'utilisateur de faire un choix
        $Choice = Read-Host "Entrez le numéro de l'opération"

        switch ($Choice) {
            "1" { Add-User }
            "2" { Add-Group }
            "3" { Add-OU }
            "4" { Write-Host "Au revoir !" -ForegroundColor Yellow; break }
            default { Write-Host "Option invalide. Essayez à nouveau." -ForegroundColor Red }
        }
        
        # Pause avant de revenir au menu principal
        if ($Choice -ne "4") { Read-Host "Appuyez sur Entrée pour revenir au menu" }
        
    } while ($Choice -ne "4")
}
```