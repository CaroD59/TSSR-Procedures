
# Installation de Guacamole sur un serveur Linux

## 1. Mise à jour et installation des dépendances

Exécutez les commandes suivantes pour mettre à jour votre système et installer les dépendances nécessaires :

```bash
apt-get update && apt-get upgrade
apt-get install -y build-essential 
apt-get install -y libcairo2-dev 
apt-get install -y libjpeg-turbo8-dev 
apt-get install -y libpng-dev 
apt-get install -y libtool-bin 
apt-get install -y libossp-uuid-dev 
apt-get install -y libvncserver-dev 
apt-get install -y freerdp2-dev 
apt-get install -y libssh2-1-dev 
apt-get install -y libtelnet-dev 
apt-get install -y libwebsockets-dev 
apt-get install -y libpulse-dev 
apt-get install -y libvorbis-dev
apt-get install -y libwebp-dev 
apt-get install -y libssl-dev 
apt-get install -y libpango1.0-dev 
apt-get install -y libswscale-dev 
apt-get install -y libavcodec-dev 
apt-get install -y libavutil-dev 
apt-get install -y libavformat-dev
```

## 2. Installation de Guacamole Server

Téléchargez et installez Guacamole Server en suivant les étapes ci-dessous :

```bash
wget https://downloads.apache.org/guacamole/1.5.3/source/guacamole-server-1.5.3.tar.gz -O /tmp/guacamole-server.tar.gz 2>/dev/null;
tar -xzf /tmp/guacamole-server.tar.gz -C /tmp/;
cd /tmp/guacamole-server-*
./configure --with-init-dir=/etc/init.d --disable-guacenc;
make # -$(nproc) 1>/dev/null;
make install;
ldconfig;
systemctl daemon-reload; # Rafraîchir la liste des services
systemctl start guacd; # Démarrage du service Guacamole
systemctl enable guacd; # Démarrage automatique au démarrage
systemctl status guacd; # Vérifier le statut du service Guacamole
mkdir -p /etc/guacamole/{extensions,lib};
```

## 3. Configuration de Tomcat

Ajoutez le dépôt Debian Bullseye pour installer Tomcat 9, puis téléchargez et installez le fichier WAR de Guacamole :

```bash
echo " deb http://deb.debian.org/debian/ bullseye main " > /etc/apt/sources.list.d/bullseye.list

# Installation de Tomcat 9
apt-get update
apt-get install -y tomcat9 
apt-get install -y tomcat9-admin 
apt-get install -y tomcat9-common
apt-get install -y tomcat9-user 

# Téléchargement et déploiement de Guacamole WAR
wget https://downloads.apache.org/guacamole/1.5.3/binary/guacamole-1.5.3.war -O /tmp/guacamole.war;
mv /tmp/guacamole.war /var/lib/tomcat9/webapps/guacamole.war;

# Redémarrage des services
systemctl restart tomcat9 guacd;
```

## 4. Installation et configuration de MariaDB

Installez MariaDB et configurez la base de données pour Guacamole :

```bash
apt-get install -y mariadb-server

mysql -u root -e " DROP DATABASE IF EXISTS guacadb; "
mysql -u root -e " CREATE DATABASE IF NOT EXISTS guacadb; "
mysql -u root -e " DROP USER IF EXISTS 'Guacamole'@'localhost'; "
mysql -u root -e " CREATE USER 'Guacamole'@'localhost' IDENTIFIED BY 'mypassword'; "
mysql -u root -e " GRANT ALL PRIVILEGES ON guacadb.* TO 'Guacamole'@'localhost'; "

sed -i 's/^/#/' /etc/apt/sources.list.d/bullseye.list
sed -i -e " s/127.0.0.1/0.0.0.0/g " /etc/mysql/mariadb.conf.d/50-server.cnf; 
systemctl restart mariadb;
```

## 5. Installation des extensions et du connecteur JDBC

Téléchargez et installez l'extension JDBC de Guacamole pour la gestion des utilisateurs :

```bash
wget https://downloads.apache.org/guacamole/1.5.3/binary/guacamole-auth-jdbc-1.5.3.tar.gz -O /tmp/guacamole-auth-jdbc.tar.gz;
tar -xzf /tmp/guacamole-auth-jdbc.tar.gz -C /tmp/;
mv /tmp/guacamole-auth-jdbc-1.5.3/mysql/guacamole-auth-jdbc-mysql-1.5.3.jar /etc/guacamole/extensions/;

# Téléchargement et installation du connecteur JDBC MySQL
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-j-8.2.0.tar.gz -O /tmp/mysql-connector-j-8.2.0.tar.gz;
tar -xvf /tmp/mysql-connector-j-8.2.0.tar.gz -C /tmp;
mv /tmp/mysql-connector-j-8.2.0/mysql-connector-j-8.2.0.jar /etc/guacamole/lib;
```

## 6. Configuration des fichiers de Guacamole

Modifiez les fichiers de configuration pour spécifier les paramètres de connexion à la base de données MySQL :

```bash
mysql -u root guacadb < /tmp/guacamole-auth-jdbc-1.5.3/mysql/schema/001-create-schema.sql;
mysql -u root guacadb < /tmp/guacamole-auth-jdbc-1.5.3/mysql/schema/002-create-admin-user.sql

# Configuration de guacd
echo " [server]" >>  /etc/guacamole/guacd.conf
echo "bind_host = 0.0.0.0 " >>  /etc/guacamole/guacd.conf
echo "bind_port = 4822" >> /etc/guacamole/guacd.conf

# Configuration de la base de données dans Guacamole
echo "mysql-hostname: 127.0.0.1" >>  /etc/guacamole/guacamole.properties
echo "mysql-port: 3306" >>  /etc/guacamole/guacamole.properties
echo "mysql-database: guacadb" >>  /etc/guacamole/guacamole.properties
echo "mysql-username: Guacamole" >>  /etc/guacamole/guacamole.properties
echo "mysql-password: mypassword" >>  /etc/guacamole/guacamole.properties
```

## 7. Finalisation

Redémarrez les services pour appliquer toutes les modifications :

```bash
systemctl restart tomcat9 guacd mariadb;
```

L'installation de Guacamole est maintenant terminée ! Vous pouvez accéder à l'interface web en vous rendant sur `http://<votre_serveur>:8080/guacamole`.
