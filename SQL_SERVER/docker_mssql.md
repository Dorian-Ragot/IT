# CONTENERISATION SQL SERVER
Cette documentation à pour but de permettre la contenerisation d'une base de donnée sql server.

## Environnement de travail
* GNU/Linux Debian 10 - Buster
* Docker version 18.09.1
* sql server version 2017
* DBeaver version 21.0.1

## Configuration de l'environnement host
Dans un premier temps nous allons préparer l'environnement host qui vas permettre de d'acceuillir le conteneur qui hébergera notre base de données sql serveur.


Nous commençons par mettre à jours notre systeme
```shell
sudo apt update && sudo apt dist-upgrade
```

Puis, nous installons l'outils Docker
```shell
sudo apt install docker.io
```

Une fois installé, nous pouvons vérifier son bon fonctionnement en affichant la version de Docker que nous utilisons.

```shell
sudo docker version
```
```
Client:
 Version:           18.09.1
 API version:       1.39
 Go version:        go1.11.6
 Git commit:        4c52b90
 Built:             Sun, 21 Feb 2021 18:18:35 +0100
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.09.1
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.11.6
  Git commit:       4c52b90
  Built:            Sun Feb 21 17:18:35 2021
  OS/Arch:          linux/amd64
  Experimental:     false
```
Nous remarquons ici que nous somme en : 18.09.1

Nous allons donc pouvoir réccupérer l'image Docker pour sql server.
```shell
sudo docker pull mcr.microsoft.com/mssql/server:2017-latest
```
Nous pouvons vérifier que nous avons la bonne image Docker.
```shell
sudo Docker images ls 
```
```shell
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
mcr.microsoft.com/mssql/server   2017-latest         861938177a03        7 weeks ago         1.3GB
```
Enfin, nous pouvons instencier notre conteneur Docker.
```shell
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=YourStrong@Passw0rd" -p 1433:1433 --name sql1 -h sql1 -d mcr.microsoft.com/mssql/server:2019-latest
```

| commande  | rôles   |
|---|---|
|  docker run | permet de lancer le conteneur  |
| -e "ACCEPT_EULA=Y"  | permet d'accepter les licences micro$oft   |
|  -e "SA_PASSWORD=YourStrong@Passw0rd" |  défini le mot de passe pour le compte SA sur la bdd (SA qui est le compte administrateur) |
|-p 1433:1433| defini que le port 1433 de l'hote est attibué au conteneur (1433 est le port par défaut pour sql server)|
|--name sql1| défini le nom du conteneur |
|-h sql1| permet de définir le hostname du conteneur|
|-d mcr.microsoft.com/mssql/server:2019-latest| permet de lancer l'image suivante en mode démon|

## Connection à la BDD
Une fois lancé, nous allons pouvoir nous connecter à la BDD avec l'outil DBeaver.

    DBeaver est un logiciel de Management de BDD open-source qui prend en compte l'ensemble des langages et des BDD

Pour installer DBeaver vous pouvez vous rendre sur le lien suivant :
* https://dbeaver.io/download/

Une fois installé, il suffit de lancer DBeaver et de rentrer les informations pour se connecter à la BDD.

Un tutoriel est disponnible ici :
* https://www.numelion.com/dbeaver-gestion-de-bases-de-donnees.html

### Tester la connection à la BDD

Pour tester la connection, il suffit de renter une commande sql dans DBeaver

```sql
SELECT name FROM sys.sysdatabase
GO
```

### Note

Il est possible de se connecter à une BDD distante avec DBeaver via un tunnel SSH.