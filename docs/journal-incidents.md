markdown
# Journal des incidents

Conformément au cahier des charges (page 9), ce journal documente les difficultés rencontrées, les diagnostics effectués et les solutions appliquées.

---

## Semaine 1 (24 - 28 Février 2026) - Lot 1 FreeIPA

---

### Incident 1 - Problème de droits sudo sur Debian (srv-infra)
**Date** : 26/02/2026  
**Responsable** : Binôme B  
**Lot concerné** : Lot 1 - Préparation VM

**Description** :
Après l'installation de Debian 12 sur srv-infra, l'utilisateur créé n'a pas les droits sudo. Les commandes `sudo` échouent avec le message "n'est pas dans le fichier sudoers".

**Diagnostic** :
```bash
sudo apt update
# Message : "hawaa n'est pas dans le fichier sudoers"

**Cause** : 
Par défaut, Debian n'ajoute pas l'utilisateur au groupe sudo lors de l'installation (contrairement à Ubuntu). C'est une mesure de sécurité, mais qui nécessite une configuration manuelle.

**Solution** 

bash
su -                    # Passer à l'utilisateur root (mot de passe root)
usermod -aG sudo hawaa  # Ajouter l'utilisateur au groupe sudo
exit                    # Quitter la session root
# Reconnection, sudo fonctionne désormais

**Statut** : ✅ Résolu
Preuve : screenshots/lot1-freeipa/incident001-sudo-fixed.png
```



### Incident 2 - Impossible d'éditer /etc/hosts sans sudo (srv-web)
**Date** : 26/02/2026  
**Responsable** : Binôme B  
**Lot concerné** : Lot 1 - Configuration réseau

**Description** :

En essayant de modifier le fichier /etc/hosts avec nano, l'erreur "Permission denied" s'est affichée. Impossible de sauvegarder les modifications apportées au fichier.

**Diagnostic** :
bash
nano /etc/hosts
Après modification, tentative de sauvegarde :
Error writing /etc/hosts: Permission denied
ls -l /etc/hosts
-rw-r--r-- 1 root root 345 Feb 27 10:30 /etc/hosts

**cause** :
Le fichier /etc/hosts appartient à root et n'est pas modifiable par un utilisateur standard. Les permissions (644) permettent la lecture par tous mais l'écriture uniquement par root. L'oubli de sudo est une erreur courante chez les administrateurs débutants.

**solution** :
bash
sudo nano /etc/hosts

**Édition réussie avec les droits root**

**Statut** : ✅ Résolu
**Preuve** : screenshots/lot1-freeipa/incident002-sudo-hosts.png


## SEMAINE 3 :(09 - 13 Mars 2026) — Lots 4 & 5

### Incident 6 — Blocage du proxy sortant inopérant (Lot 4)
**Date** : 09/03/2026    
**Lot concerné** : Lot 4 — Services Web, Proxy et FTP

**Description** :
Dans le cadre de la configuration du proxy sortant sur Apache (port 8080), un problème majeur a été constaté : le blocage des réseaux sociaux ne fonctionnait pas. La commande de test :
```bash
curl -x http://localhost:8080 http://facebook.com -I

retournait systématiquement un code HTTP 200 OK au lieu du 403 Forbidden attendu, signifiant que le proxy ne filtrait pas les requêtes comme configuré.

## Diagnostic :


# 1. Vérification de la configuration du proxy
cat /etc/apache2/sites-available/proxy.conf
La configuration semblait syntaxiquement correcte, avec la directive ProxyBlock bien présente :

apache
ProxyBlock facebook.com twitter.com instagram.com youtube.com
bash

# 2. Liste des sites activés
ls -la /etc/apache2/sites-enabled/
Résultat :

text
001-www.conf -> ../sites-available/001-www.conf
002-intranet.conf -> ../sites-available/002-intranet.conf
003-webmail.conf -> ../sites-available/003-webmail.conf
proxy.conf -> ../sites-available/proxy.conf
bash

# 3. Analyse approfondie de l'ordre des VirtualHosts
sudo apache2ctl -S
Cette commande a révélé le problème :

text
port 8080 namevhost www.smarttech.sn (/etc/apache2/sites-enabled/001-www.conf:1)
port 8080 namevhost intranet.smarttech.sn (/etc/apache2/sites-enabled/002-intranet.conf:1)
port 8080 namevhost webmail.smarttech.sn (/etc/apache2/sites-enabled/003-webmail.conf:1)
port 8080 namevhost proxy.smarttech.sn (/etc/apache2/sites-enabled/proxy.conf:1)

##Cause 
Apache traite les VirtualHosts dans l'ordre alphabétique des fichiers présents dans sites-enabled/. Le fichier 001-www.conf étant chargé avant proxy.conf, toutes les requêtes sur le port 8080 sans en-tête Host spécifique étaient dirigées vers le site www.smarttech.sn au lieu du proxy. Le blocage ProxyBlock n'était donc jamais appliqué.

##Solution :
Renommer le fichier du proxy pour le rendre prioritaire :

sudo mv /etc/apache2/sites-available/proxy.conf /etc/apache2/sites-available/000-proxy-default.conf

##Désactiver l'ancien fichier et activer le nouveau :

sudo a2dissite proxy.conf
sudo a2ensite 000-proxy-default.conf


##Vérification de l'ordre des VirtualHosts :
sudo apache2ctl -S | grep -A5 "port 8080"

##Résultat après correction :

port 8080 namevhost default-proxy.smarttech.sn (/etc/apache2/sites-enabled/000-proxy-default.conf:1)
port 8080 namevhost www.smarttech.sn (/etc/apache2/sites-enabled/001-www.conf:1)
...
##Rechargement de la configuration Apache :

sudo systemctl reload apache2

##Test final du blocage :

curl -x http://localhost:8080 http://facebook.com -I

##Résultat attendu et obtenu :

HTTP/1.1 403 Forbidden

##Vérification dans les logs :

sudo tail -f /var/log/apache2/proxy-access.log

Les logs confirment le blocage :

127.0.0.1 - - [09/Mar/2026:01:05:45 +0000] "HEAD http://facebook.com/ HTTP/1.1" 403 140 "-" "curl/7.81.0"

##Résultat :
Le blocage des réseaux sociaux fonctionne désormais parfaitement. Toutes les requêtes passant par le proxy sur le port 8080 sont correctement filtrées, retournant un code 403 pour les sites interdits.
