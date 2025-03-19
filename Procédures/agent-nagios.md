# Script d'installation de NRPE

---

### Mise à jour du système et installation des prérequis

```bash
apt-get update  
apt-get install -y autoconf automake gcc libc6 libmcrypt-dev make libssl-dev wget  
dpkg -L libssl-dev  
```

### Téléchargement de NRPE

```bash
wget --no-check-certificate -O nrpe.tar.gz https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-4.1.3/nrpe-4.1.3.tar.gz  
```

### Extraction et installation de NRPE

```bash
tar xzf nrpe.tar.gz  
cd nrpe-4.1.3  
./configure --enable-command-args  
make all  
make install-groups-users  
make install && make install-config  
```

### Vérification de la ligne "nrpe 56666" dans /etc/services

```bash
echo "Test si la ligne contenant nrpe 56666 existe"  
echo "Alors ne rien faire et continuer votre installation"  
echo "Sinon, il ajouter ce qui suit :"  
echo "echo >> /etc/services"  
echo "echo '# Nagios services' >> /etc/services"  
echo "echo 'nrpe 5666/tcp' >> /etc/services"  
```

### Installation et configuration du service NRPE

```bash
make install-init  
systemctl enable nrpe.service  
```

### Installation de iptables-persistent et ajout de la règle du firewall

```bash
read p  
echo "installation du iptables-persistent...."  
apt install -y iptables-persistent  
echo "Ajout de la règle du firewall"  
iptables -I INPUT -p tcp --destination-port 5666 -j ACCEPT  
echo "Vérification si la règle du parefeu a été bien ajoutée"  
cat /etc/iptables/rules.v4  
echo "Alors vous l'avez vu la règle du parefeu ??"  
```

### Modification du fichier de configuration NRPE

```bash
echo "Modifier le fichier suivant"  
echo "/usr/local/nagios/etc/nrpe.cfg"  
echo "en modifiant allowed_hosts=172.16.20.128, avec votre adresse du Serveur Nagios"  
echo "aussi il faut modifier : dont_blame_nrpe=1"  
```

### Démarrage du service NRPE

```bash
echo "Démarrage du serveur NRPE"  
systemctl start nrpe.service  

service nrpe status  
echo "Est-ce que le statut de votre daemon est ok ?? tout est Vert ??"  
echo "Sinon, appeler votre instructeur"  
/usr/local/nagios/libexec/check_nrpe -H localhost  
echo "Si vous avez le message suivant au dessus : NRPE v4.1.3, alors C'est déjà pas mal"
```