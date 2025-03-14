# Installation d'un Active Directory en mode Core

### Entrée dans la console PowerShell
```powershell
powershell
```

---

#### Si réalisé sur VMware, Installer les VMTools :
```powershell
D:\setup.exe
```

---

### Renommer le serveur
```powershell
Rename-Computer -NewName AD-M2I-1
Restart-Computer -Force
$env:COMPUTERNAME
```

---

### Désactiver le pare-feu
```powershell
netsh advfirewall set allprofiles state off
```

---

### Ajouter les services AD-DS
```powershell
Add-WindowsFeature AD-Domain-Services`=
```

---

### Ajouter le domaine
```powershell
Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath "C:\Windows\ntds" -DomainMode "Win2012" -DomainName "M2i.local" -DomainNetbiosName "M2i" -ForestMode "Win2012" -InstallDns:$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:$false -SysvolPath "C:\Windows\SYSVOL" -Force:$true
```

Notes:  
Un mot de passe vous sera demandé, ici "Azerty1" pour l'exemple, mettez ce que vous voulez.  
> Secure mdp demandé : Azerty1

---

### Changer l'adresse IP du Windows Server
```powershell
netsh interface ipv4 set address name= Ethernet0 static 192.168.10.1 255.255.255.0
```

Notes:  
Mettez l'adresse IP, l'interface ainsi que le masque correspondant à votre infrastructure.

---

### Ajouter le service DHCP
Installation du service :
```powershell
Install-WindowsFeature -name dhcp -includemanagementtools
```

Configuration du service :
```powershell
Add-DhcpServerInDC -DnsName AD1 192.168.10.1
Add-DhcpServerv4Scope -Name "M2i" -StartRange 192.168.10.10 -EndRange 192.168.10.50 -SubnetMask 255.255.255.0
Set-DhcpServerv4OptionDefinition -OptionId 3 -DefaultValue 192.168.10.254
Set-DhcpServerv4OptionDefinition -OptionId 6 -DefaultValue 192.168.10.1
Set-DhcpServerv4OptionDefinition -OptionId 15 -DefaultValue M2i.local
```

Notes:  
Modifiez les informations selon votre infrastructure.

---

### Vérifier que tout est installé
```powershell
Get-WindowsFeature -Name DHCP
Get-WindowsFeature -Name AD-Domain-Services
```

---

### Vérifier les informations des services Active Directory
```powershell
Get-ADDomainController
Get-ComputerInfo | Select-Object CsDomain, CsDomainRole
```

---

### Vérifier l'étendue DHCP
```powershell
Get-DhcpServerv4Scope
```

---

### Création des unités d'organisation
```powershell
New-ADOrganizationalUnit -Name "Stagiaire" -Path "DC=M2i,DC=local"
New-ADOrganizationalUnit -Name "Formateur" -Path "DC=M2i,DC=local"
New-ADOrganizationalUnit -Name "Administration" -Path "DC=M2i,DC=local"
New-ADOrganizationalUnit -Name "TSSR" -Path "OU=Stagiaire,DC=M2i,DC=local"
New-ADOrganizationalUnit -Name "WEB-design" -Path "OU=Stagiaire,DC=M2i,DC=local"
```

---

### Création des comptes utilisateurs
```powershell
New-ADUser -Name User1 -GivenName User1 -Surname "User1" -SamAccountName "User.1" -UserPrincipalName "User.1" -CannotChangePassword $true -PasswordNeverExpires $true -AccountPassword (ConvertTo-SecureString -AsPlainText -force "Azerty1") -ChangePasswordAtLogon $false -Enabled $true -Organization "WEB-design"
New-ADUser -Name User2 -GivenName User2 -Surname "User2" -SamAccountName "User.2" -UserPrincipalName "User.2" -CannotChangePassword $true -PasswordNeverExpires $true -AccountPassword (ConvertTo-SecureString -AsPlainText -force "Azerty1") -ChangePasswordAtLogon $false -Enabled $true -Organization "WEB-design"
`New-ADUser -Name User3 -GivenName User3 -Surname "User3" -SamAccountName "User.3" -UserPrincipalName "User.3" -CannotChangePassword $true -PasswordNeverExpires $true -AccountPassword (ConvertTo-SecureString -AsPlainText -force "Azerty1") -ChangePasswordAtLogon $false -Enabled $true -Organization "WEB-design"
```

---

### Création des groupes
```powershell
`New-ADGroup -Name "G1" -DisplayName "G1" -GroupCategory security -GroupScope global -path "OU=Formateur,DC=M2i,DC=local"
`New-ADGroup -Name "G2" -DisplayName "G2" -GroupCategory distribution -GroupScope universal -path "OU=Administration,DC=M2i,DC=local"
```

---

### Supprimer l'utilisateur User1
```powershell
`Remove-ADUser -Identity "User.1"
```

Notes:  
Validez avec "O".

---

### Désactiver l'utilisateur User2
```powershell
Disable-ADAccount -Identity "User.2"
```

---

### Ajouter l'utilisateur User3 au groupe G1
```powershell
Add-ADGroupMember -Identity "G1" -Members "User.3"
```

---

### Affecter l'utilisateur User2 au groupe G2
```powershell
Add-ADGroupMember -Identity "G2" -Members "User.2"
```

---

### Activer un compte désactivé
```powershell
Enable-ADAccount -Identity "User.2"
```

---

## Création d'utilisateurs depuis un fichier CSV

Création du fichier contenant les profils :
```powershell
New-Item -Name "Stagiaire_TSSR.csv" -Path "C:\" -ItemType File
```

Ouverture pour modification :
```powershell
notepad.exe C:\Stagiaire_TSSR.csv
```

Création des utilisateurs :
```powershell
Import-Csv "C:\Stagiaire_TSSR.csv" -Delimiter "," | New-ADUser -CannotChangePassword $false -ChangePasswordAtLogon $true -PasswordNeverExpires $false -Enabled $true
```

---

### Supprimer l'unité d'organisation WEB-design
Retirer la protection contre la suppression :
```powershell
Get-ADOrganizationalUnit -Identity "OU=WEB-design,OU=Stagiaire,DC=M2i,DC=local" | Set-ADOrganizationalUnit -ProtectedFromAccidentalDeletion$false
```

Supprimer l'unité d'organisation :
```powershell
Remove-ADOrganizationalUnit -Identity "OU=WEB-design,OU=Stagiaire,DC=M2i,DC=local"
```

Notes:  
Validez avec "O".

---

### Afficher tous les objets de l'AD
```powershell
- `Get-ADObject -Filter "*"`
```

---

## Script permettant de créer un utilisateur, un groupe ou une unité d'organisation

```powershell
$user_choice = Read-Host "Tapez 1 pour créer un utilisateur, 2 pour créer un groupe, 3 pour créer une unité d'organisation"
switch ($user_choice) {
    1 {
        $Surname = Read-Host -Prompt "Entrez le nom : "
        $Givename = Read-Host -Prompt "Entrez le prénom :"
        $mdp = Read-Host -AsSecureString "Entrez le mot de passe :"
        $SamaccountName = $Givename.Substring(0,1) + "." + $Surname
        $name = $Givename + " " + $Surname

        New-ADUser -Name $name -GivenName $Givename -Surname $Surname -SamAccountName $SamaccountName -UserPrincipalName $SamaccountName -CannotChangePassword $true -PasswordNeverExpires $true -AccountPassword $mdp -ChangePasswordAtLogon $false -Enabled $true 

        Write-Host "Utilisateur créé avec succès"
    }
    2 {
        $Group_Name = Read-Host -Prompt "Entrez le nom du groupe : "
        New-ADGroup -Name $Group_Name -DisplayName $Group_Name -GroupCategory Security -GroupScope Global

        Write-Host "Le groupe a bien été créé"
    }
    3 {
        $UO_name = Read-Host -Prompt "Entrez le nom de l'unité d'organisation :"
        $chemin = Read-Host -Prompt "Indiquez le chemin où l'unité d'organisation doit être créée :"
        New-ADOrganizationalUnit -Name $UO_name -Path $chemin

        Write-Host "L'unité d'organisation a bien été créée"
    }
}
```