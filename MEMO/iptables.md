# IPTABLES MEMO

## Définition
iptables est un logiciel libre de l'espace utilisateur Linux grâce auquel l'administrateur système peut configurer les chaînes et règles dans le pare-feu en espace noyau (et qui est composé par des modules Netfilter).

Différents programmes sont utilisés selon le protocole employé : iptables est utilisé pour le protocole IPv4, Ip6tables pour IPv6, Arptables pour ARP (Address Resolution Protocol) ou encore Ebtables, spécifique aux trames Ethernet.

Ce type de modifications doit être réservé à un administrateur du système. Par conséquent, son utilisation nécessite l'utilisation du compte root. L'utilisation du programme est refusée aux autres utilisateurs.

## Fonctionnement
iptables permet de créer des tableaux, qui contiennent des "chaînes", elles-mêmes composées d'un ensemble de règles de traitement des paquets.

Chaque paquet réseaux, entrant ou sortant, traverse donc au moins une chaîne.

Il existe cinq chaînes prédéfinies, cependant leur utilisation n'est pas obligatoire dans chaque tableau. Les chaînes prédéfinies ont une politique, qui est appliquée au paquet s'il arrive à la fin de la chaîne.

Les cinq types de chaînes prédéfinies sont les suivants : 
* "PREROUTING" : Les paquets vont entrer dans cette chaîne avant qu'une décision de routage ne soit prise.
* "INPUT" : Le paquet va être livré sur place (N.B. : la livraison sur place ne dépend pas d'un processus ayant un socket ouvert ; la livraison est contrôlée par le tableau « local » de routage : ip route show table local).
* "FORWARD" : Tous les paquets qui ont été acheminés et ne sont pas livrés sur place parcourent la chaîne.
* "OUTPUT" : Les paquets envoyés à partir de la machine elle-même se rendront à cette chaîne.
* "POSTROUTING" : La décision de routage a été prise. Les paquets entrent dans cette chaîne, juste avant qu'ils ne soient transmis vers le matériel.

Iptables comprend quatre tables nommées : mangle, filter, nat, raw.

Cet aspect est souvent mal expliqué (ou peu mis en exergue) dans les tutoriaux qui de ce fait laissent penser qu'il existe des chaînes en dehors des tables. Cette confusion vient du fait que la table filter est la table par défaut. Ainsi elle est souvent implicite dans les commandes et le discours. Par exemple lorsqu'un tutoriel parle de la chaîne INPUT sans autre précision il s'agit donc de la chaîne INPUT de la table filter. 

## Configuration
### Lister les tables
```shell
sudo iptables -L
```
```shell
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

### Réinitialiser une modification précédente
```shell
sudo iptables -F
```
```shell
sudo iptables -X
```

### Ouvrir un port spécifique
Ici, nous allons ouvrir le port 22 (port par défaut de SSH)
```shell
sudo iptables -A INPUT -p tcp -i eth0 --dport 22 -j ACCEPT
```
| Argument | Explication          |
| :--------------- |:---------------|
| iptables  | Appel de la commande  |
| -A INPUT | Ajoute une règle à la chaîne INPUT pour le trafic entrant   |
| -p tcp | Définis le protocole tcp |
| -i eth0  | Défini l'interface ethernet |
| --dport 22 | Défini le port 22 (peut être remplacer par --dport ssh) |
| -j ACCEPT | Authorise le trafic |

### Supprimer une règle
D'abord on liste les règles :
```shell
iptables -L --line-numbers
```
```shell
Chain INPUT (policy DROP)
num  target     prot opt source               destination
1    DROP       icmp --  anywhere             anywhere
2    ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:ssh
3    ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:www
4    ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:webmin

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     tcp  --  anywhere             anywhere            tcp spt:www
2    ACCEPT     tcp  --  anywhere             anywhere            tcp spt:12345
```

Si on souhaite supprimer la règle pour authoriser le ssh (ligne 2)
```shell
iptables -D INPUT 2
```
| Argument | Explication          |
| :--------------- |:---------------|
| iptables  | Appel de la commande  |
| -D INPUT | Supprime la règle de la chaîne INPUT |
| 2  | Défini la ligne a supprimer |

## Sauvegarder les règles au démarrage

### Via iptables-persistent

```shell
service iptables-persistent
```

### Via un fichier (systemd)

TO DO