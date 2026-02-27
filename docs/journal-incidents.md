# Journal des incidents

Conformément au cahier des charges (page 9), ce journal documente les difficultés rencontrées, les diagnostics effectués et les solutions appliquées.

---

## Semaine 1 (24 - 28 Février 2026) - Lot 1 FreeIPA

### Incident 1 - Problème de droits sudo sur Debian (srv-infra)
**Date** : 26/02/2026  
**Responsable** : Binôme B  
**Lot concerné** : Lot 1 - Préparation VM

**Description** :
Après l'installation de Debian 12 sur srv-infra, l'utilisateur créé n'a pas les droits sudo. Les commandes `sudo` échouent avec le message "n'est pas dans le fichier sudoers".

**Diagnostic** :
```bash
sudo apt update
# Message : "hawaa n'est pas dans le fichier sudoers".


##Cause :
Par défaut, Debian n'ajoute pas l'utilisateur au groupe sudo lors de l'installation (contrairement à Ubuntu). C'est une mesure de sécurité, mais qui nécessite une configuration manuelle.

##su -                    # Passer à l'utilisateur root (mot de passe root)
usermod -aG sudo nawaa  # Ajouter l'utilisateur au groupe sudo
exit                    # Quitter la session root
# Reconnection, sudo fonctionne désormais



---


---

### Incident 2 - Impossible d'éditer /etc/hosts sans sudo


### Incident 5 - Impossible d'éditer /etc/hosts sans sudo
**Date** : 27/02/2026  
**Responsable** : Binôme B  
**Lot concerné** : Configuration réseau

**Description** :
En essayant de modifier le fichier `/etc/hosts` avec nano, l'erreur "Permission denied" s'est affichée.

**Diagnostic** :
```bash
nano /etc/hosts
# Error writing /etc/hosts: Permission denied
ls -l /etc/hosts
# -rw-r--r-- 1 root root ... /etc/hosts


### Cause :
Le fichier /etc/hosts appartient à root et n'est pas modifiable par un utilisateur standard. L'oubli de sudo est une erreur courante.

### Solution

bash
sudo nano /etc/hosts
# Édition réussie avec les droits root


