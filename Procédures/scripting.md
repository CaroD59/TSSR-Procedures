# Script de création et d'exécution de fichiers

### Créer un script avec `nano`

1. Ouvre un fichier script avec `nano` :
    ```bash
    nano fichier.sh
    ```
2. Mets-y le contenu suivant :
    ```bash
    #!/bin/bash
    
    echo "text"
    
    ls -l
    
    chmod u+x fichier
    
    ls -l
    
    cat fichier
    
    ./fichier
    ```
    Ce script affiche un message "text", liste les fichiers du répertoire, donne les permissions d'exécution à `fichier`, liste à nouveau les fichiers, affiche le contenu de `fichier`, et enfin, l'exécute.

3. Pour ceux qui veulent créer un script plus facilement, copiez-collez ce contenu dans un autre script pour le rendre exécutable :
    ```bash
    #!/bin/bash
    touch $1
    chmod 744 $1
    echo "#!/bin/bash" >> $1
    nano $1
    ```
    Ce script crée un fichier avec un nom spécifié en argument (`$1`), lui donne les permissions `744`, puis y ajoute la ligne `#!/bin/bash` et ouvre le fichier avec `nano` pour l'éditer.

4. Pour rendre un script exécutable :
    ```bash
    chmod u+x fichier.sh
    ```

5. Pour exécuter le script :
    ```bash
    ./fichier.sh
    ```

---

# Configuration du serveur DHCP

1. Installe le serveur DHCP avec `apt-get` :
    ```bash
    sudo apt-get install isc-dhcp-server
    ```

2. Modifie les paramètres réseau avec `nano` :
    ```bash
    sudo nano /etc/network/interfaces
    ```
    Ajoute ou modifie la configuration réseau :
    ```text
    auto ens33
    iface ens33 inet static
        address 192.168.200.100
        netmask 255.255.255.0
        gateway 192.168.200.254
    ```

3. Sauvegarde le fichier de configuration DHCP :
    ```bash
    sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.sav
    ```

4. Modifie le fichier de configuration DHCP :
    ```bash
    sudo nano /etc/dhcp/dhcpd.conf
    ```
    Exemple de configuration pour un sous-réseau `192.168.200.0` :
    ```text
    subnet 192.168.200.0 netmask 255.255.255.0 {
        range 192.168.200.10 192.168.200.50;
        option subnet-mask 255.255.255.0;
        option domain-name-servers 8.8.8.8;
        option domain-name "m2i.local";
        option routers 192.168.200.254;
        option broadcast-address 192.168.200.255;
        default-lease-time 600;
        max-lease-time 7200;
    }
    ```

5. Modifie le fichier `/etc/resolv.conf` pour spécifier les serveurs DNS :
    ```bash
    sudo nano /etc/resolv.conf
    ```
    Exemple de contenu :
    ```text
    domain m2i.local
    search m2i.local
    nameserver 192.168.200.100
    nameserver 192.168.200.254
    ```

6. Configuration dans VMware :
    - Serveur DHCP : Allez dans **Virtual Network Editor**, sélectionnez `vmnet8`, cliquez sur **Change Settings**, puis décochez l'option "Use local DHCP service."
    - Client : Clic droit -> **Settings** -> **Network Adapter** -> **Custom** -> Sélectionnez `vmnet8 (NAT)`.

7. Redémarre le service DHCP :
    ```bash
    sudo systemctl restart isc-dhcp-server
    ```

---

# Configuration DNS

1. Installe `dnsutils` pour tester les configurations DNS :
    ```bash
    sudo apt-get install dnsutils
    ```

2. Modifie le fichier `/etc/hostname` pour définir le nom de l'hôte :
    ```bash
    sudo nano /etc/hostname
    ```
    Exemple :
    ```text
    srv-lx01.m2i.local
    ```

3. Modifie les interfaces réseau dans `/etc/network/interfaces` pour ajouter les serveurs DNS :
    ```bash
    sudo nano /etc/network/interfaces
    ```
    Ajoute :
    ```text
    dns-nameservers 192.168.200.100
    dns-domain m2i.local
    ```

4. Modifie `/etc/hosts` pour lier les adresses IP aux noms d'hôtes :
    ```bash
    sudo nano /etc/hosts
    ```
    Exemple de contenu :
    ```text
    127.0.0.1       localhost
    127.0.1.1       srv-lx01.m2i.local       srv-lx01
    192.168.200.100 srv-lx01.m2i.local       srv-lx01
    192.168.200.102 srv-lx01.m2i.local       srv-lx02
    192.168.200.103 srv-lx01.m2i.local       srv-lx03
    192.168.200.104 srv-lx01.m2i.local       srv-lx04
    192.168.200.105 srv-lx01.m2i.local       srv-lx05
    ```

5. Modifie `/etc/resolv.conf` pour définir les serveurs DNS :
    ```bash
    sudo nano /etc/resolv.conf
    ```
    Exemple de contenu :
    ```text
    domain m2i.local
    search m2i.local
    nameserver 192.168.200.100
    nameserver 192.168.200.254
    ```

6. Installe le serveur DNS `bind9` :
    ```bash
    sudo apt-get install bind9
    ```

---

# Services supplémentaires

### Apache

Pour installer et configurer Apache, vous pouvez utiliser les commandes suivantes :
```bash
sudo apt-get install apache2
```

### MariaDB / MySQL
Pour installer MariaDB (ou MySQL), utilisez :

```bash
sudo apt-get install mariadb-server
```

### WordPress
Pour installer WordPress, vous devrez installer Apache, PHP, et MySQL (ou MariaDB), puis télécharger et configurer WordPress :

```bash
sudo apt-get install wordpress
```