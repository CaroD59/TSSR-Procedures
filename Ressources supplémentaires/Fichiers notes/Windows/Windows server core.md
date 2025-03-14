# installation d'un active directory en mode core

### entré dans la console de powershell
	powershell

-------------------------------------------------------------------
#### si réalisé sur VMWare, Installer les VMTools :
	D:\setup.exe

-------------------------------------------------------------------
### renomé le serveur
	- Rename-Computer -NewName AD-M2I-1
	- Restart-Computer -Force
	- $env:COMPUTERNAME

-------------------------------------------------------------------
### Désactiver pare-feu :
	netsh advfirewall set allprofiles state off

-------------------------------------------------------------------
### Ajout AD-DS Services :
- Add-WindowsFeature AD-Domain-Services

## Ajout domaine :
	Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath "C:\Windows\ntds" -DomainMode "Win2012" -DomainName "M2i.local" -DomainNetbiosName "M2i" -ForestMode "Win2012" -InstallDns:$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:$false -SysvolPath "C:\Windows\SYSVOL" -Force:$true

Notes:
un mot de passe vous sera demandé, ici Azerty1 pour l'exemple; mettez ce que vous voulez
>Secure modp demandé: Azerty1

-------------------------------------------------------------------
### Changer l'adresse IP du windows server :
	netsh interface ipv4 set address name= Ethernet0 static 192.168.10.1 255.255.255.0

Notes:
> mettez l'adresse IP, l'interface ainsi que le masque correspondant à votre infrastructure

-------------------------------------------------------------------
### Ajout du service DHCP :
installation du service:
	
	Install-WindowsFeature -name dhcp -includemanagementtools

configuration du service:
		
	Add-DhcpServerInDC -DnsName AD1 192.168.10.1
	Add-DhcpServerv4Scope -Name "M2i" -StartRange 192.168.10.10 -EndRange 192.168.10.50 -SubnetMask 255.255.255.0
	Set-DhcpServerv4OptionDefinition -OptionId 3 -DefaultValue 192.168.10.254
	Set-DhcpServerv4OptionDefinition -OptionId 6 -DefaultValue 192.168.10.1
	Set-DhcpServerv4OptionDefinition -OptionId 15 -DefaultValue M2i.local

Notes:
> modifié les informations selon votre infrastructure

-------------------------------------------------------------------
### Vérifier que tout est installé :
	Get-WindowsFeature -Name DHCP
	Get-WindowsFeature -Name AD-Domain-Services

-------------------------------------------------------------------
### Vérifier les informations des services Active Directory :
	Get-ADDomainController
	Get-ComputerInfo | Select-Object CsDomain, CsDomainRole

-------------------------------------------------------------------
### Vérifier l'étendu DCHP :
	Get-DhcpServerv4Scope
	
-------------------------------------------------------------------
### Création des organisation :
	New-ADOrganizationalUnit -Name "Stagiaire" -Path "DC=M2i,DC=local"
	New-ADOrganizationalUnit -Name "Formateur" -Path "DC=M2i,DC=local"
	New-ADOrganizationalUnit -Name "Administration" -Path "DC=M2i,DC=local"
	New-ADOrganizationalUnit -Name "TSSR" -Path "OU=Stagiaire,DC=M2i,DC=local"
	New-ADOrganizationalUnit -Name "WEB-design" -Path "OU=Stagiaire,DC=M2i,DC=local"

-------------------------------------------------------------------
### création des comptes utilisateurs :
	New-ADUser -Name User1 -GivenName User1 -Surname "User1" -SamAccountName "User.1" -UserPrincipalName "User.1" -CannotChangePassword $true -PasswordNeverExpires $true -AccountPassword (ConvertTo-SecureString -AsPlainText -force "Azerty1") -ChangePasswordAtLogon $false -Enabled $true -Organization "WEB-design"
	New-ADUser -Name User2 -GivenName User2 -Surname "User2" -SamAccountName "User.2" -UserPrincipalName "User.2" -CannotChangePassword $true -PasswordNeverExpires $true -AccountPassword (ConvertTo-SecureString -AsPlainText -force "Azerty1") -ChangePasswordAtLogon $false -Enabled $true -Organization "WEB-design"
	New-ADUser -Name User3 -GivenName User3 -Surname "User3" -SamAccountName "User.3" -UserPrincipalName "User.3" -CannotChangePassword $true -PasswordNeverExpires $true -AccountPassword (ConvertTo-SecureString -AsPlainText -force "Azerty1") -ChangePasswordAtLogon $false -Enabled $true -Organization "WEB-design"

-------------------------------------------------------------------
### création des groupes :
	New-ADGroup -Name "G1" -DisplayName "G1" -GroupCategory security -GroupScope global -path "OU=Formateur,DC=M2i,DC=local"
	New-ADGroup -Name "G2" -DisplayName "G2" -GroupCategory distribution -GroupScope universal -path "OU=Administration,DC=M2i,DC=local"

-------------------------------------------------------------------
### supprimer le user1 :
	Remove-ADUser -Identity "User.1"

Notes: 
> Valider avec "O"
-------------------------------------------------------------------
### désactiver le user2 :
	Disable-ADAccount -Identity "User.2"

-------------------------------------------------------------------
### ajouter le user3 au groupe g1 :
	Add-ADGroupMember -Identity "G1" -Members "User.3"

-------------------------------------------------------------------
### affecter user2 au groupe g2 :
	Add-ADGroupMember -Identity "G2" -Members "User.2"

-------------------------------------------------------------------
### activé un compte ayant était désactivé:
	Enable-ADAccount -Identity "User.2"

-------------------------------------------------------------------
## création d'utilisateur depuis un csv :


creation du fichier contenant les profils:

	New-Item -Name "Stagiaire_TSSR.csv" -Path "C:\" -ItemType File

ouverture pour modification:

	notepad.exe C:\Stagiaire_TSSR.csv 

création des utilisateurs:

	Import-Csv "C:\Stagiaire_TSSR.csv" -Delimiter "," | New-ADUser -CannotChangePassword $false -ChangePasswordAtLogon $true -PasswordNeverExpires $false -Enabled $true

-------------------------------------------------------------------
### supprimer l'unité d'organisation WEB-design :

retiré la protection contre la suppression

	Get-ADOrganizationalUnit -Identity "OU=WEB-design,OU=Stagiaire,DC=M2i,DC=local" | Set-ADOrganizationalUnit -ProtectedFromAccidentalDeletion$false

supprimer l'unité d'organisation :

	Remove-ADOrganizationalUnit -Identity "OU=WEB-design,OU=Stagiaire,DC=M2i,DC=local"
	
Notes:
> Validation avec "O"

-------------------------------------------------------------------
### afficher tout les objet de l'AD :
	Get-ADObject -Filter "*"


## script qui permet de crée un utilisateur, un groupe ou une unité d'organisation

	$user_choice = Read-Host "taper 1 pour crée un utilisateur, 2 pour crée un groupe, 3 pour crée une unité d'organisation"
	switch ($user_choice) {
    1 {
        $Surname = Read-Host -Prompt "entre le nom: "
        $Givename = Read-Host -Prompt "entre le prenom:"
        $mdp = Read-Host -AsSecureString "entre le mot de passe:"
        $SamaccountName = $Givename.Substring(0,1) + "." + $Surname
        $name = $Givename + " " + $Surname

        New-ADUser -Name $name -GivenName $Givename -Surname $Surname -SamAccountName $SamaccountName -UserPrincipalName $SamaccountName -CannotChangePassword $true -PasswordNeverExpires $true -AccountPassword $mdp -ChangePasswordAtLogon $false -Enabled $true 
    
        Write-Host "utilisateur cree avec succes"
    }
    2 {
        $Group_Name = Read-Host -Prompt "entre le nom du groupe: "
        New-ADGroup -Name $Group_Name -DisplayName $Group_Name -GroupCategory Security -GroupScope Global
    
        Write-Host "le groupe a bien etait cree"
    }
    3 {
        $UO_name = Read-Host -Prompt "entre le nom de l'unite d'organisation:"
        $chemin = Read-Host -Prompt "indiquer le chemin ou l'unite d'organisation doit etre cree:"
        New-ADOrganizationalUnit -Name $UO_name -path $chemin
        
        Write-Host "l'unite d'organisation a bien etait cree"
    }
	}
