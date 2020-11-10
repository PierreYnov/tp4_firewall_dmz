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

### - Installation de Apache :

```
sudo yum install httpd
```

On ouvre ensuite le port 80 pour qu'on puisse se connecter au HTTP/HTTPS :

```
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

Ensuite on peut démarrer le serveur httpd :

```
sudo systemctl start httpd
sudo systemctl enable httpd
```
On peut vérifier son status :

![](img/httpd.png)

> Le service est bien démarré. Il suffit de se rendre sur internet pour vérifier notre page en allant sur l'adresse IP de notre machine : http://192.168.43.89 

![](img/page_apache.png)

### - Installation de wordpress :

Pour l'installation de wordpress, avoir préalablement PHP d'installer avec une version supérieur à 5.6 pour que ce soit fonctionnelle.

- Téléchargement et installation de wordpress :

```
cd ~
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
sudo rsync -avP ~/wordpress/ /var/www/html/
mkdir /var/www/html/wp-content/uploads
sudo chown -R apache:apache /var/www/html/*
```

- Configuration de la base de donnée pour wordpress :

```
cd /var/www/html
cp wp-config-sample.php wp-config.php
vim wp-config.php
```

Fichier de conf (remplacer par les infos correspondantes):

![](wp-config.png)

Apache ne lis pas les fichiers PHP par défault, donc dans le fichiervi /etc/httpd/conf/httpd.conf, il faut ajouter à Directory index :

``index.php ``

On redémarre le serveur Httpd (sudo service httpd restart), et on remarque que le wordpress est mis en place. 

### Iptables

#### Désactiver FirewallD

    sudo systemctl stop firewalld
    sudo systemctl disable firewalld
    sudo systemctl mask --now firewalld

#### Installation et Activation de Iptables

    sudo yum install iptables-services
    sudo systemctl start iptables
    sudo systemctl start ip6tables
    sudo systemctl enable iptables
    sudo systemctl enable ip6tables

### Check du service & des règles appliquées

    sudo systemctl status iptables
    sudo iptables -nvL


### Filtrage Iptables

    sudo vi /etc/sysconfig/iptables
On y ajoute ces 4 lignes avant les 2 lignes finissant par prohibited

``-A INPUT -m state --state NEW -p tcp --dport  80  -j ACCEPT``

``-A INPUT -m state --state NEW -p tcp --dport  443  -j ACCEPT``

 ``-A INPUT -m state --state NEW -m tcp -p tcp -s ip_serveur_bdd --dport 3306 -j ACCEPT``

 ``-A OUTPUT -m state --state NEW -m tcp -p tcp -s ip_serveur_bdd --dport 3306 -j ACCEPT``


Sauvegarder puis

    sudo service iptables restart

Pour check

    sudo iptables -L

## 3.1.3 Préparation du serveur de base de données :

Le serveur de base de données devra être positionné sur un réseau dédié comme indiqué sur le schéma, et disposer d’une adresse IP fixe. Il fonctionnera sous Centos 8 , et hébergera via un serveur mysql la base de données du site Wordpress.

### - Installation du server Mysql sur centos8 :

```
sudo dnf install mysql-server
sudo systemctl start mysqld.service
sudo systemctl enable mysqld.service
```

On peut vérifier ensuite l'installation par un status :

![](img/mysql-server.png)

> Le serveur mysql est bien démarré. 

### - Création de la base de données wordpress :

```
CREATE DATABASE wordpress;
```

- Création d'un utilisateur admin et ajout de privilèges :

```
CREATE USER 'adminuser'@'localhost' IDENTIFIED BY 'toortoor';
GRANT ALL PRIVILEGES ON wordpress. * TO 'adminuser'@'localhost';
FLUSH PRIVILEGES;
```

### Nftables



### Désactivation de FirewallD

    sudo systemctl disable --now firewalld
    sudo systemctl mask firewalld
    reboot
    
### Ajout port

    sudo nft add table inet filter
    sudo nft add chain inet filter INPUT
    sudo nft add chain inet filter OUTPUT
    sudo nft add chain inet filter FORWARD

    sudo nft add rule inet filter INPUT tcp dport 22 accept
    sudo nft add rule inet filter INPUT tcp dport 80 accept
    sudo nft add rule inet filter INPUT tcp dport 443 accept
    sudo nft add rule inet filter INPUT ip protocol icmp accept

    sudo nft add rule inet filter OUTPUT tcp dport 3306 ip daddr ip_server_web accept

Garder ces règles dans un fichier pour pouvoir les recharger après un reboot

    sudo nft list ruleset > myrules

Pour remettre

    sudo nft -f myrules

Pour les garder en permanent

    nft list ruleset > /etc/sysconfig/nftables.conf
    systemctl enable nftables.service
    reboot

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
    