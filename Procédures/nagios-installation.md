# Script : Installation automatique de Nagios Server

## Pour Linux Distribution : Debian ou Ubuntu

---

## Avant d'exécuter ce script assurez-vous d'avoir :
- Une connexion internet avec résolution de nom
- Le script doit être exécuté avec les droits root

---

### Mise à jour du Système

```bash
apt-get update -y && apt-get upgrade -y
```

### Installation des prérequis pour Nagios

```bash
apt-get install wget build-essential unzip openssl libssl-dev apache2 php libapache2-mod-php php-gd libgd-dev -y
```

### Vérification de l'installation des prérequis

```bash
echo -e "L'installation des prérequis s'est terminée en succès ?? (O/n) \c"
read reponse
case $reponse in
    O|o|0)  
        echo "Nous allons continuer l'installation ;-) "
        ;;
    N|n)  
        echo "Veuillez faire les vérifications nécessaires avant de relancer ce script"
        exit 1
        ;;
    *)  
        echo "Choix non autorisé ... Au revoir"
        exit 1
        reboot -t 0
        ;;
esac
```

### Ajout de l'utilisateur 'nagios' et liaison avec les groupes Apache et nagios

```bash
adduser nagios
groupadd nagcmd
usermod -a -G nagcmd nagios
usermod -a -G nagcmd www-data
```

### Téléchargement et Installation de Nagios Core Service

```bash
cd /opt/
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.5.9.tar.gz
tar xzf nagios-4.5.9.tar.gz
```

### Extraction, Compilation et Configuration de Nagios

```bash
cd nagios-4.5.9
./configure --with-command-group=nagcmd
make all
make install
make install-init
make install-config
make install-commandmode

cp -R contrib/eventhandlers/ /usr/local/nagios/libexec/
chown -R nagios:nagios /usr/local/nagios/libexec/eventhandlers
```

### Mise en place et paramétrage de l'Authentication avec Apache

```bash
echo "ScriptAlias /nagios/cgi-bin \"/usr/local/nagios/sbin\"" > /etc/apache2/conf-available/nagios.conf
echo " " >> /etc/apache2/conf-available/nagios.conf
echo "<Directory \"/usr/local/nagios/sbin\">" >> /etc/apache2/conf-available/nagios.conf
echo "   Options ExecCGI " >> /etc/apache2/conf-available/nagios.conf
echo "   AllowOverride None " >> /etc/apache2/conf-available/nagios.conf
echo "   Order allow,deny " >> /etc/apache2/conf-available/nagios.conf
echo "   Allow from all " >> /etc/apache2/conf-available/nagios.conf
echo '   AuthName "Restricted Area" ' >> /etc/apache2/conf-available/nagios.conf
echo "   AuthType Basic " >> /etc/apache2/conf-available/nagios.conf
echo "   AuthUserFile /usr/local/nagios/etc/htpasswd.users " >> /etc/apache2/conf-available/nagios.conf
echo "   Require valid-user " >> /etc/apache2/conf-available/nagios.conf
echo "</Directory> " >> /etc/apache2/conf-available/nagios.conf
echo " " >> /etc/apache2/conf-available/nagios.conf
echo "Alias /nagios \"/usr/local/nagios/share\" " >> /etc/apache2/conf-available/nagios.conf
echo " " >> /etc/apache2/conf-available/nagios.conf
echo "<Directory \"/usr/local/nagios/share\">" >> /etc/apache2/conf-available/nagios.conf
echo "   Options None " >> /etc/apache2/conf-available/nagios.conf
echo "   AllowOverride None " >> /etc/apache2/conf-available/nagios.conf
echo "   Order allow,deny " >> /etc/apache2/conf-available/nagios.conf
echo "   Allow from all " >> /etc/apache2/conf-available/nagios.conf
echo "   AuthName 'Restricted Area' " >> /etc/apache2/conf-available/nagios.conf
echo "   AuthType Basic " >> /etc/apache2/conf-available/nagios.conf
echo "   AuthUserFile /usr/local/nagios/etc/htpasswd.users " >> /etc/apache2/conf-available/nagios.conf
echo "   Require valid-user " >> /etc/apache2/conf-available/nagios.conf
echo "</Directory> " >> /etc/apache2/conf-available/nagios.conf
```

### Attribution du mot de passe pour : nagios

```bash
read mot2passe
echo "votre mot de passe est :$mot2passe" 
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

### Configuration du site nagios sur le serveur Apache2

```bash
a2enconf nagios
a2enmod cgi rewrite
service apache2 restart

echo "Appuyez sur une touche pour continuer "
read choix
```

---

## Installation des Plugins Nagios

```bash
cd /opt
wget https://github.com/nagios-plugins/nagios-plugins/releases/download/release-2.4.12/nagios-plugins-2.4.12.tar.gz
tar xzf nagios-plugins-2.4.12.tar.gz
cd nagios-plugins-2.4.12
```

```bash
read  choix
```

### Compilation des Plugins Nagios

```bash
read choix
./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl
make
make install
```

---

## Vérifications des paramètres

```bash
read choix
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
echo "Si vous n'avez aucune erreur alors l'installation est achevée en succès"
```

### Démarrage du service Nagios

```bash
service nagios start
systemctl enable nagios
read choix
```

---

## Tester maintenant si votre serveur Nagios est installé correctement !
- Utilisez la commande `service nagios status` ou `/etc/init.d/nagios status`
