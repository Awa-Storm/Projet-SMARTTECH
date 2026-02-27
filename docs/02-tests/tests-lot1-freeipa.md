# Tests - Lot 1 : FreeIPA-Machine B
## Tests préparatoires réalisés le 27/02/2026


### Test P.1 : Vérification des adresses IP

**Sur srv-web (192.168.10.40)** :
```bash
$ ip a | grep inet
    inet 127.0.0.1/8 scope host lo
    inet 192.168.10.40/24 brd 192.168.10.255 scope global enp0s3

**Sur srv-mail (192.168.10.50)** :
$ ip a | grep inet
    inet 127.0.0.1/8 scope host lo
    inet 192.168.10.50/24 brd 192.168.10.255 scope global enp0s3

**Sur srv-infra (192.168.10.10)** :
$ ip a | grep inet
    inet 127.0.0.1/8 scope host lo
    inet 192.168.10.10/24 brd 192.168.10.255 scope global enp0s3

Résultat : ✅ Toutes les IP sont correctement configurées



### Test P.2 : Vérification du fichier /etc/hosts
Sur chaque VM :
$ cat /etc/hosts
127.0.0.1       localhost
192.168.10.10   srv-infra.smarttech.sn    srv-infra
192.168.10.20   srv-ipa.smarttech.sn      srv-ipa
192.168.10.30   srv-share.smarttech.sn    srv-share
192.168.10.40   srv-web.smarttech.sn      srv-web
192.168.10.50   srv-mail.smarttech.sn     srv-mail
192.168.10.100  client.smarttech.sn       client

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

Résultat : ✅ Toutes les machines sont listées dans /etc/hosts


### Test P.3 : Connectivité réseau entre les VMs
Résultat : ✅ Connectivité réseau OK entre toutes les VMs

### Test P.4 : Installation du client FreeIPA
Résultat : ✅ Client FreeIPA installé sur les 3 VMs


# Bilan des prérequis validés
VM	                 IP	      /etc/hosts	 Connectivité	  Client FreeIPA	
srv-web	       192.168.10.40	   ✅	         ✅      	✅	
srv-mail	     192.168.10.50	   ✅	         ✅       	✅	
srv-infra	     192.168.10.10	   ✅          	 ✅      	✅	
Tous les prérequis sont validés. En attente du serveur FreeIPA (binôme A) pour procéder à la jonction au domaine.








