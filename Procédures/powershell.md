# PowerShell

## Installer VMTools :
```powershell
Get-PSDrive
cd d:
ls
.\setup.exe
```
---

## Machine 1
```powershell
# Renommer le nom de l'ordinateur :
    Rename-Computer -NewName AD-M2I-1
# Redémarrer l'ordinateur :
    Restart-Computer -Force
# Vérifier le nom de l'ordinateur :
    $env:COMPUTERNAME
```
---

### Désactiver le pare-feu :
```powershell
netsh advfirewall set allprofiles state off
```

---

### Ajouter les services ADDS :
```powershell
Add-WindowsFeature AD-Domain-Services
```
---

### Ajouter un domaine :
```powershell
Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath "C:\Windows\ntds" -DomainMode "Win2012" -DomainName "m2i-core.local" -DomainNetbiosName "m2i-core" -ForestMode "Win2012" -InstallDns:$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:$false -SysvolPath "C:\Windows\SYSVOL" -Force:$true
```

---

### Changer l'adresse IP :
```powershell
netsh interface ipv4 set address name= Ethernet0 static 192.168.10.1 255.255.255.0
```

---

### Ajouter DHCP :
```powershell
Install-WindowsFeature -name dhcp -includemanagementtools
Add-DhcpServerInDC -DnsName AD-M2I-1 192.168.10.1
Add-DhcpServerv4Scope -Name "M2I-Core" -StartRange 192.168.10.10 -EndRange 192.168.10.50 -SubnetMask 255.255.255.0
`Set-DhcpServerv4OptionDefinition -OptionId 3 -DefaultValue 192.168.10.254
`Set-DhcpServerv4OptionDefinition -OptionId 6 -DefaultValue 192.168.10.1
`Set-DhcpServerv4OptionDefinition -OptionId 15 -DefaultValue M2I-Core.local
```
---

### Vérifier que tout est installé :
```powershell
Get-WindowsFeature -Name DHCP
Get-WindowsFeature -Name AD-Domain-Services
```

---

### Vérifier les informations :
```powershell
Get-ADDomainController
Get-ComputerInfo | Select-Object CsDomain, CsDomainRole
```

---

### Vérifier l'étendue DHCP :
```powershell
Get-DhcpServerv4Scope
```

---

## Machine 2 (Serveur) :

- Lancer PowerShell :
```powershell
powershell
```

---

### Renommer l'ordinateur :
```powershell
Rename-Computer -NewName AD-M2I-1-Server
# Redémarrer l'ordinateur :
Restart-Computer (-force)
# Vérifier le nom de l'ordinateur :
$env:COMPUTERNAME
```
---

### Désactiver le pare-feu :
```powershell
netsh advfirewall set allprofiles state off
```

---

### Changer l'adresse IP :
```powershell
netsh interface ipv4 set address name= Ethernet0 static 192.168.10.2 255.255.255.0
```

---

### Joindre la machine au domaine :
```powershell
# Pour connaître l'index de l'interface, exécuter :
Get-Netadapter
# Définir l'adresse du serveur DNS :
Set-DnsClientServerAddress -InterfaceIndex 3 -ServerAddresses 192.168.10.1
# Ajouter l'ordinateur au domaine :
Add-Computer -ComputerName AD-M2I-1-Server -DomainName M2I-Core.local
```

---

### /!\ Si cela ne fonctionne pas :

- Vérifier si on peut pinguer le serveur : 
```powershell
ping 192.168.10.1
```
- Vérifier la configuration DNS :
```powershell
nslookup M2I-Core.local
```
- Vérifier que les adresses IP sont dans le même réseau !
- Redémarrer l'ordinateur.

---

### Ajouter les services ADDS :
```powershell
Add-WindowsFeature AD-Domain-Services
# Redémarrer l'ordinateur :
Restart-Computer (-force)
```

---

### Faire échapper deux fois lorsque le mot de passe est demandé, sélectionner "autre utilisateur" et remplir les informations.

---

### Installer le contrôleur de domaine supplémentaire :
```powershell
$password = ConvertTo-SecureString -AsPlainText -String Azerty1 -force
```

- Installer le contrôleur de domaine :
```powershell
Install-ADDSDomainController -DomainName M2I-Core.local -DatabasePath "C:\Windows\NTDS" -LogPath "C:\Windows\NTDS" -SysvolPath "C:\Windows\SYSVOL" -InstallDns -ReplicationSourceDC AD-M2I-1.M2i-core.local -SafeModeAdministratorPassword $password -NoRebootOnCompletion
```
- Redémarrer l'ordinateur :
```powershell
Restart-Computer (-force)
```
