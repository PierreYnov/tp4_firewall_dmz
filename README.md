# TP4 : Utilisation avancé du filtrage de paquets réseaux
## Construction d’une DMZ complexe



Création de 2 DMZ :

- publique avec le serveur web
- privée avec la bdd


![](https://i.gyazo.com/ffacc0d9f17f25c05104f4333be5885a.png)

Config de règles de filtrage sur des serv linux avec iptables et nftables + 2 firewalls pfsense



## 3.1.1 Considérations générales :

- tout l'entrant / sortant est bloqué
- s'assure que le firewall soit en stateful

## 3.1.2 Préparation du serveur Web :

Centos 7

Apache
Wordpress
Ip fixe

config filtrage avec ip tables

- update autorisé
- peut communique serveur BDD
- possible de s'y connecter depuis partout
- ssh
- ping


## 3.1.3 Préparation du serveur de base de données :

Centos 9
mysql du Wordpress

config filtrage avec nftables

- update aurtorisé
- consultable que par le serv web
- ssh
- ping


## 3.2.1 Préparation du firewall frontal 

TCP 80 et 443 du Frontal vers le serv Web

- sur le pfsense :
    
    internet + local peut aller sur le site
    ssh
    ping
    navigation internet

## 3.2.2 Préparation du firewall de seconde ligne 

- sur le pfsense :

    internet + local peut aller sur le site
    ssh
    webgui firewall
    ping
    navigation internet 
    