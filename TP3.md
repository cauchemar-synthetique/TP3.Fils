#Prologue
### I. Configurer GNS3
**On configure les machines comme suit :**
```
$ sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s8
NOM=enp0s3, enp0s8 ou enp0s9
DEVICE=enp0s3, enp0s8 ou enp0s9

BOOTPROTO=statique
ONBOOT=oui

IPADDR=ip Réseau 1, réseau 2 ou réseau 3
NETMASK=255.255.255.0 ou 255.255.255.254 pour le réseau 3
```
### II. Itinéraires itinéraires itinéraires
**🌞 Activer le routage sur les deux machines router**

**Sur le routeur 1 et 2**
```
$ sudo sysctl -w net.ipv4.ip_forward=1
$ sudo pare-feu-cmd --add-masquerade
$ sudo pare-feu-cmd --add-masquerade --permanent
```
**🌞 Mettre en place les routes locales**
```
$ cat /etc/sysconfig/network-scripts/route-enp0s8
10.3.2.0/24 via 10.3.100.2
$ sudo ip route ajouter 10.3.2.0/24 via 10.3.100.2 dev enp0s8
```
```
$ sudo ip route ajouter 10.3.1.0/24 via 10.3.100.1 dev enp0s3

$ sudo cat /etc/sysconfig/network-scripts/route-enp0s3
10.3.1.0/24 via 10.3.100.1

$ping 10.3.2.11
PING 10.3.2.11 (10.3.2.11) 56(84) octets de données.
64 octets de 10.3.2.11 : icmp_seq=1 ttl=64 time=1,38 ms
```
**🌞 Mettre en place les routes par défaut**
```
$ echo 'Passerelle=10.3.1.254' | sudo tee /etc/sysconfig/network
$ echo 'Passerelle=10.3.1.254' | sudo tee /etc/sysconfig/network
$ echo 'Passerelle=10.3.2.254' | sudo tee /etc/sysconfig/network
$ echo 'Passerelle=10.3.2.254' | sudo tee /etc/sysconfig/network
$ echo 'Passerelle=10.3.100.1' | sudo tee /etc/sysconfig/network
```
**Depuis node1.net1.tp3**
```
$ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) octets de données.
64 octets de 8.8.8.8 : icmp_seq=6 ttl=113 time=17,2 ms
```
```
$ traceroute 1.1.1.1
traceroute vers 1.1.1.1 (1.1.1.1), 30 sauts maximum, paquets de 60 octets
 1 * * *
 2 192.168.122.1 (192.168.122.1) 1,232 ms 1,193 ms 1,430 ms
 3 10.100.0.1 (10.100.0.1) 6,530 ms 6,494 ms 6,459 ms
 4 10.100.255.11 (10.100.255.11) 5,474 ms 5,433 ms 5,183 ms
 5 185.176.176.10 (185.176.176.10) 23,785 ms 23,741 ms 23,622 ms
 6 100.126.127.254 (100.126.127.254) 18,039 ms 17,725 ms 17,682 ms
 7 100.126.127.253 (100.126.127.253) 16,172 ms 14,681 ms 14,629 ms
 8 185.181.155.200 (185.181.155.200) 14,796 ms 17,744 ms 17,698 ms
 9 linktsas-ic-381495.ip.twelve99-cust.net (62.115.186.121) 17,659 ms 18,032 ms 16,854 ms
10 prs-b1-link.ip.twelve99.net (62.115.186.86) 16,807 ms prs-b9-link.ip.twelve99.net (62.115.186.120) 16,772 ms 15,291 ms
11 cloudflare-ic-375100.ip.twelve99-cust.net (80.239.194.103) 15,426 ms prs-b1-link.ip.twelve99.net (62.115.115.88) 19,741 ms 18,157 ms
12 cloudflare-ic-363840.ip.twelve99-cust.net (213.248.73.69) 18,101 ms .128.2)
20,049 ms
13 172.71.116.2 (172.71.116.2) 37,892 ms un.un.un.un (1.1.1.1) 19,711 ms 172.71.132.2 (172.71.132.2) 19,592 ms
```

# I. Serveur DHCP
### I. Configuration ```dhcp.net1.tp3```
```
$ sudo ip route ajouter la valeur par défaut via 10.3.1.254 dev enp0s3
$ sudo dnf install -y serveur-dhcp
```
```
$ sudo cat /etc/dhcp/dhcpd.conf

durée de location par défaut 3 600 ;
durée de location maximale 86 400 ;
faisant autorité;

sous-réseau 10.3.1.0 masque de réseau 255.255.255.0 {
plage 10.3.1.50 10.3.1.99 ;
option masque de sous-réseau 255.255.255.0 ;
routeurs d'options 10.3.1.254 ;
option serveurs de noms de domaine 1.1.1.1 ;
}
```
🌞 **Preuve**
```
$ip un
2 : enp0s3 : <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel état UP groupe par défaut qlen 1000
    lien/éther 08:00:27:da:11:76 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.50/24 brd 10.3.1.255 portée globale dynamique noprefixroute enp0s3
       valid_lft 3515sec préféré_lft 3515sec
    inet6 fe80 :: a00: 27ff: feda: lien de portée 1176/64
       valid_lft pour toujours préféré_lft pour toujours
```
```
$iprs
par défaut via 10.3.1.254 dev enp0s3 proto dhcp src 10.3.1.50 métrique 100
10.3.1.0/24 dev enp0s3 proto kernel scope link src 10.3.1.50 métrique 100
192.168.56.0/24 dev enp0s8 proto noyau portée lien src 192.168.56.2 métrique 101
```
```
$ spectacle de développement nmcli | grep 'IP4.DNS'
IP4.DNS[1] : 1.1.1.1
```

#II. Serveur WEB
### I.Installation
🌞 **Installation du serveur web NGINX**
```
$ sudo dnf install -y nginx
```
### 2. Page HTML et Racine WEB
**Page HTML de Création d'une bête**
```
$ sudo mkdir /var/www/fils
$ sudo chown nginx:nginx fils/
$ touch /var/www/fils/index.html && nano /var/www/fils/index.html
BIENVENUE Cauchemar_sythétique
```
### 3. Configuration de NGINX
**🌞 Création d'un fichier de configuration NGINX**
```
$ sudo touch /etc/nginx/conf.d/web.net2.tp3.conf && sudo nano /etc/nginx/conf.d/web.net2.tp3.conf
```
```
  serveur {
      nom_serveur web.net2.tp3 ;
      écoutez 10.3.2.101:80 ;

      emplacement / {
          racine /var/www/fils;
          index index.html;
      }
  }
```
### 4. Pare-feu
**🌞 Ouvrir le port nécessaire dans le pare-feu**
```
$ sudo pare-feu-cmd --add-port=80/tcp --permanent
```
```
$ sudo pare-feu-cmd --reload
```
```
$ sudo pare-feu-cmd --list-all
public (actif)
  cible : par défaut
  icmp-block-inversion : non
  interfaces : enp0s3 enp0s8
  sources:
  services : cockpit dhcpv6-client ssh
  ports : 80/tcp
  protocoles :
  en avant : oui
  mascarade : non
  ports avant :
  ports source :
  blocs icmp :
  règles riches :
```
### 5. Tester
**🌞 Démarrez le service NGINX !**
```
$ sudo systemctl démarrer nginx
$ sudo systemctl activer nginx
Lien symbolique créé /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.

$ sudo systemctl statut nginx
● nginx.service - Le serveur HTTP et proxy inverse nginx
     Chargé : chargé (/usr/lib/systemd/system/nginx.service ; activé ; prédéfini : désactivé)
     Actif : actif (en cours d'exécution) depuis le samedi 07/10/2023 16:46:28 CEST ; il y a 48 ans
   PID principal : 11709 (nginx)
      Tâches : 2 (limite : 4611)
     Mémoire : 2,0 M
        Processeur : 10 ms
     Groupe C : /system.slice/nginx.service
             ├─11709 "nginx : processus maître /usr/sbin/nginx"
             └─11710 "nginx : processus de travail"
```
**🌞 Testez local**
```
$ boucle http://10.3.2.101
BIENVENUE Cauchemar_synthétique
```
**🌞 Accéder au site web depuis un client**

sur interface graphique, on arrive sur une page blanche avec :
```
BIENVENUE Cauchemar_synthétique
```
Silence sac d'os.

**🌞 Avec un nom ?**
```
$ sudo chat /etc/hosts
10.3.2.101 web.net2.tp3
```
```
$ boucle http://web.net2.tp3
BIENVENUE Cauchemar_synthétique
```
Respirez profondement

#III. Serveur DNS
### 1. Installer
```
$ sudo dnf mise à jour -y
$ sudo dnf install -y bind bind-utils
```
### 2. Configuration
```
$ sudo cat /etc/named.conf
options {
        port d'écoute 53 { 127.0.0.1 ; n'importe lequel; } ;
        port d'écoute sur v6 53 { ::1; } ;
        répertoire "/var/named" ;
        autoriser-requête { localhost ; n'importe lequel; } ;
        permettre-requête-cache { localhost ; n'importe lequel; } ;

        récursivité oui ;
[...]
zone "net2.tp3" DANS {
     tapez maître ;
     fichier "net2.tp3.db" ;
     autoriser la mise à jour { aucun ; } ;
     autoriser-requête {tout; } ;
} ;
zone "2.3.10.in-addr.arpa" IN {
     tapez maître ;
     fichier "net2.tp3.rev" ;
     autoriser la mise à jour { aucun ; } ;
     autoriser-requête { n'importe lequel ; } ;
} ;
```
**➜ Et pour les fichiers de zone**
```
$ sudo cat /var/named/net2.tp3.db
$ TTL 86 400
@ Cauchemar dns.net2.tp3. admin.net2.tp3. (
    2019061800 ; Série
    3600 ; Actualiser
    1800 ; Réessayer
    604800 ; Expire
    86400 ; Durée de vie minimale
)
@ Cauchemar dns.net2.tp3.
DNS DANS UN 10.3.2.102
web DANS UN 10.3.2.101

```
```
$ sudo cat /var/named/net2.tp3.rev

$ TTL 86 400
@ Cauchemar dns.net2.tp3. admin.net2.tp3. (
    2019061800 ; Série
    3600 ; Actualiser
    1800 ; Réessayer
    604800 ; Expire
    86400 ; Durée de vie minimale
)

@ Cauchemar dns.net2.tp3.
102 DANS PTR dns.net2.tp3.
101 DANS PTR web.net2.tp3.
```
**➜ Une fois ces 3 fichiers en place, on démarre le service DNS**
```
$ sudo systemctl start nommé
$ sudo systemctl activer nommé
$ sudo systemctl statut nommé
$ sudo journalctl -xe -u nommé
```
### 3. Pare-feu
**🌞 Ouvrir le port nécessaire dans le pare-feu**
```
$ sudo pare-feu-cmd --add-port=53/udp --permanent
succès
```
```
$ sudo pare-feu-cmd --list-all
public (actif)
  cible : par défaut
  icmp-block-inversion : non
  interfaces : enp0s3 enp0s8
  sources:
  services : cockpit dhcpv6-client ssh
  ports : 53/udp
  protocoles :
  en avant : oui
  mascarade : non
  ports avant :
  ports source :
  blocs icmp :
  règles riches :
```
### 4. Tester
**🌞 Depuis l'une des machines clientes du réseau 1 (par exemple node1.net1.tp3)**
```
$ creuser web.net2.tp3 @10.3.2.102

; <<>> DiG 9.16.23-RH <<>> web.net2.tp3 @10.3.2.102
;; options globales : +cmd
;; J'ai la réponse :
;; ->>HEADER<<- opcode : QUERY, statut : NOERROR, identifiant : 27675
;; drapeaux : qr aa rd ra ; REQUÊTE : 1, RÉPONSE : 1, AUTORITÉ : 0, SUPPLÉMENTAIRE : 1

;; OPTER LA PSEUDOSECTION :
; EDNS : version : 0, drapeaux : ; UDP : 1232
; COOKIE : 8cc46a6c44559fcd010000006522f2b99e5bbf2169ff777f (bon)
;; SECTION DES QUESTIONS :
;web.net2.tp3. DANS UN

;; SECTION RÉPONSE :
web.net2.tp3. 86400 IN A 10.3.2.101

;; Temps de requête : 2 ms
;; SERVEUR : 10.3.2.102#53(10.3.2.102)
;; QUAND : dim. 08 octobre 20:19:37 CEST 2023
;; TAILLE MSG reçue : 85
```
```
$ curl web.net2.tp3
BIENVENUE Cauchemar_sythétique
```
_mdr_

### 5. DHCP mon vieil ami
**🌞 Editez la configuration du serveur DHCP sur ```dhcp.net1.tp3```**
```
$ sudo cat /etc/dhcp/dhcpd.conf
durée de location par défaut 3 600 ;
durée de location maximale 86 400 ;
faisant autorité;

sous-réseau 10.3.1.0 masque de réseau 255.255.255.0 {
plage 10.3.1.50 10.3.1.99 ;
option masque de sous-réseau 255.255.255.0 ;
routeurs d'options 10.3.1.254 ;
option serveurs de noms de domaine 10.3.2.102 ;
}
```
```
$ spectacle de développement nmcli | grep 'IP4.DNS'
IP4.DNS[1] : 10.3.2.102
```
```
$ creuser web.net2.tp3
; <<>> DiG 9.16.23-RH <<>> web.net2.tp3
;; options globales : +cmd
;; réponse :
;; ->>HEADER<<- opcode : QUERY, statut : NOERROR, identifiant : 38967
;; drapeaux : qr aa rd ra ; REQUÊTE : 1, RÉPONSE : 1, AUTORITÉ : 0, SUPPLÉMENTAIRE : 1

;; OPTER LA PSEUDOSECTION :
; EDNS : version : 0, drapeaux : ; UDP : 1232
; COOKIE : 86415b795fb36eae010000006522f5ac049e9a8b87b6d3bd (bon)
;; SECTION DES QUESTIONS :
;web.net2.tp3. DANS UN

;; SECTION RÉPONSE :
web.net2.tp3. 86400 IN A 10.3.2.101

;; Temps de requête : 1 ms
;; SERVEUR : 10.3.2.102#53(10.3.2.102)
;; QUAND : dim. 08 octobre 20:32:13 CEST 2023
;; TAILLE MSG reçue : 85
```
