# Guide d'installation de GLPI sur un serveur avec Apache, PHP et MariaDB

## Mise à jour du système

```bash
apt-get update && apt-get upgrade
```

## Installation de Apache2

```bash
apt-get install apache2 -y
```

## Vérification d'Apache2

```bash
systemctl status apache2.service
```

## Installation de PHP

```bash
apt-get install php
```

## Installation de MariaDB

```bash
apt install mariadb-server
```

## Vérification de l'installation de MariaDB

```bash
systemctl status mariadb.service
```

## Création de la base de données GLPI et de son utilisateur

```bash
mysql -u root -e "CREATE DATABASE glpi;"
mysql -u root -e "CREATE USER 'glpibdd'@'localhost' IDENTIFIED BY 'Azerty1';"
mysql -u root -e "GRANT ALL PRIVILEGES ON glpi.* TO 'glpibdd'@'localhost';"
mysql -u root -e "FLUSH PRIVILEGES;"
```

## Redémarrage de MariaDB pour appliquer les changements

```bash
systemctl restart mariadb
```

## Installation des dépendances nécessaires

```bash
apt-get install php-ldap php-imap php-apcu php-cas php-mbstring php-curl php-gd perl php-zip php-intl php-bz2 php-mysql php-xml -y
systemctl reload apache2
```

## Téléchargement de GLPI

```bash
cd /usr/src
wget https://github.com/glpi-project/glpi/releases/download/10.0.18/glpi-10.0.18.tgz
tar -zxvf glpi-10.0.18.tgz
mkdir /var/www/html/glpi
mv /usr/src/glpi/* /var/www/html/glpi/
```

## Attribution des droits à Apache sur le dossier GLPI

```bash
chown -R www-data:www-data /var/www/html/
```

## Vérification de l'état des services

```bash
systemctl status apache2
systemctl status mariadb
php -v
```

## Identifiants pour se connecter à la base de données

```bash
echo "Identifiant de GLPI pour se connecter à la base de données"
echo "Identifiant: glpibdd"
echo "Mot de passe: Azerty1"
```

## Identifiants pour se connecter à l'interface GLPI

```bash
echo "Identifiant et mot de passe pour se connecter à l'interface de GLPI"
echo "Identifiant: glpi"
echo "Mot de passe: glpi"
```

<p align="center">
  <img src="../Procédures/images/glpi.png" alt="GLPi" style="display: block; margin: auto; max-width: 100%; height: auto;"/>
</p>