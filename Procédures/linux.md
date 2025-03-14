# Commandes de base Linux :

### Affiche le répertoire de travail actuel
```powershell
pwd
```

### Liste les fichiers dans le répertoire actuel
```powershell
ls
```

### Copie des fichiers ou répertoires
```powershell
cp fichier destination
```

### Copie un répertoire de manière récursive
```powershell
cp -r fichier destination
```

### Copie en préservant les attributs du fichier
```powershell
cp -a fichier destination
```

### Renommer un fichier
```powershell
mv fichier nouveaunomfichier
```

### Éditeur de texte en ligne de commande. Pour quitter, tape `:wq` pour sauvegarder ou `:x` pour quitter sans sauvegarder
```powershell
vi 
# ou
vim
```

---

# Gestion des utilisateurs et groupes :

### Se connecter en tant que super-utilisateur (root)
```powershell
su -
```

### Ajouter un nouvel utilisateur
```powershell
adduser prenom
```

### Modifier le mot de passe d’un utilisateur
```powershell
passwd nom
```

### Ajouter un utilisateur au groupe `sudo` pour lui donner des privilèges administratifs
```powershell
usermod -aG sudo prenom
```

### Supprime un utilisateur et son répertoire personnel
```powershell
userdel -r user
```

### Affiche les groupes d’un utilisateur
```powershell
groups user
```

### Créer un groupe
```powershell
groupadd nomgroupe
```

### Supprimer un groupe
```powershell
groupdel nomgroupe
```

---

# Permissions et droits d’accès :

### Modifier les permissions des fichiers et répertoires
```powershell
chmod
```
#### Exemple : `chmod 764 fichier.txt` (lecture, écriture, exécution pour le propriétaire, lecture et écriture pour le groupe, lecture pour les autres)
##### Utilisation de l'octal : Les droits sont codés de 0 à 7, avec `r = 4`, `w = 2`, `x = 1`

### Changer le propriétaire d’un fichier ou répertoire
```powershell
chown
```
#### Exemple : `chown -R user /home/user/*` pour changer récursivement le propriétaire

### Changer le groupe propriétaire d’un fichier ou répertoire
```powershell
chgrp nomgroupe fichier
```

---

# Recherche de fichiers :

### Recherche de fichiers
```powershell
find . -name fichier
```
#### Le point signifie le répertoire actuel, sinon il faut mettre le chemin

### Recherche un répertoire nommé `document`
```powershell
find /home -type d -name document
```
### Cherche les fichiers vides
```powershell
find / -type f -empty
```
### Recherche rapide de fichiers
```powershell
- `locate` :
  - Installation : `apt install mlocate`
  - Exemple : `locate fichier`
```
---

# Gestion des packages :

### Met à jour la liste des packages
```powershell
apt-get update
```

### Met à jour les packages installés à la dernière version disponible
```powershell
apt-get upgrade
```

### Met à jour les packages et gère les dépendances lors d’une migration de version
```powershell
apt-get dist-upgrade
```

### Installer un package
```powershell
apt-get install package
```

### Désinstaller un package
```powershell
apt-get remove package
```

### Supprimer les 
#### Recherche un package par mot-clé
```powershell
apt-cache search
```

---

# Création de liens :

### Créer un lien physique
```powershell
ln
```
#### Exemple : `ln fichier destination/nouveaunomfichier`

### Créer un lien symbolique
```powershell
ln -s
```
#### Exemple : `ln -s dossier/ nouveaunom`

---

# Alias et raccourcis :

### Pour faciliter l'utilisation de commandes comme `apt-get update` et `apt-get upgrade`, tu peux créer un alias.
### Exemple : Ajouter `alias maj='apt-get update && apt-get upgrade'` dans le fichier `.bashrc` pour utiliser `maj` comme raccourci.

---

# Commandes utiles pour la sécurité et la gestion des utilisateurs :

### Affiche la liste des utilisateurs
```powershell
cat /etc/passwd
```

### Affiche la liste des groupes
```powershell
cat /etc/group
```

### Affiche les informations sensibles sur les mots de passe des utilisateurs
```powershell
cat /etc/shadow
```

---

# Autres commandes pratiques :

### Affiche l'arborescence des répertoires
```powershell
# tree
apt-get install tree
# Exemple : `tree /home`
```