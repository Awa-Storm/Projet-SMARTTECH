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


