# Chiffrement d'un disque avec LUKS 2

Cette note a pour but de permettre le chiffrement d'un disque de donnée avec l'utilitaire LUKS.

## LUKS 
LUKS, pour Linux Unified Key Setup, est le standard associé au noyau Linux pour le chiffrement de disque créé par Clemens Fruhwirth.

LUKS permet de chiffrer l'intégralité d'un disque de telle sorte que celui-ci soit utilisable sur d'autres plates-formes et distributions de Linux (voire d'autres systèmes d'exploitation). Il supporte des mots de passe multiples, afin que plusieurs utilisateurs soient en mesure de déchiffrer le même volume sans partager leur mot de passe.

## Chiffrement du disque

*l'ensemble des commandes se font avec les droits administrateur de la machine*

On commence par lister les groupes de volume de disque.
```shell
# lsblk
```
```shell
NAME              MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda                 8:0    0 136.1G  0 disk  
├─sda1              8:1    0   953M  0 part  /boot
└─sda2              8:2    0 135.2G  0 part  
  ├─vgroot-lvroot 254:0    0 111.8G  0 lvm   /
  ├─vgroot-lvswap 254:1    0   3.7G  0 lvm   [SWAP]
  ├─vgroot-lvtmp  254:2    0   9.3G  0 lvm   /tmp
  └─vgroot-lvhome 254:3    0  10.4G  0 lvm   /home
sdb                 8:16   0  14.6T  0 disk  
```
On remarque que notre partie data est le volume sdb.

Nous commençons par le partitionner avec **parted**
```shell
# parted /dev/sdb
```

Une fois notre disque partitionné, nous installons le package qui nous permettra de chiffrer le dique.
```shell
# apt install cryptsetup
```
Une fois installé, nous pouvons regarder si nous avons la dernière version disponnible.

```shell
# cryptsetup --version
```
```shell
cryptsetup 2.3.5
```
Ici, nous somme en versions 2.3.5.

Ensuite, nous allons faire un benchmark pour choisir le meilleur chiffrement. De préférence, un chiffrement puissant sans trop de perte de performance. 
```shell
# cryptsetup benchmark
```
```shell
# Tests are approximate using memory only (no storage IO).
PBKDF2-sha1       488163 iterations per second for 256-bit key
PBKDF2-sha256    1024000 iterations per second for 256-bit key
PBKDF2-sha512     733269 iterations per second for 256-bit key
PBKDF2-ripemd160  521161 iterations per second for 256-bit key
PBKDF2-whirlpool  332670 iterations per second for 256-bit key
argon2i       4 iterations, 776003 memory, 4 parallel threads (CPUs) for 256-bit key (requested 2000 ms time)
argon2id      4 iterations, 764179 memory, 4 parallel threads (CPUs) for 256-bit key (requested 2000 ms time)
#     Algorithm |       Key |      Encryption |      Decryption
        aes-cbc        128b       529.2 MiB/s      1248.8 MiB/s
    serpent-cbc        128b        33.8 MiB/s       220.1 MiB/s
    twofish-cbc        128b       108.4 MiB/s       171.0 MiB/s
        aes-cbc        256b       418.4 MiB/s       999.9 MiB/s
    serpent-cbc        256b        54.0 MiB/s       220.1 MiB/s
    twofish-cbc        256b       125.8 MiB/s       170.9 MiB/s
        aes-xts        256b      1111.6 MiB/s      1086.2 MiB/s
    serpent-xts        256b       165.5 MiB/s       213.1 MiB/s
    twofish-xts        256b       154.2 MiB/s       165.0 MiB/s
        aes-xts        512b       904.7 MiB/s       876.8 MiB/s
    serpent-xts        512b       205.8 MiB/s       213.1 MiB/s
    twofish-xts        512b       164.8 MiB/s       164.9 MiB/s
```

Ici nous remarquons que l'aes-xts en 256b est le meilleur chiffrement pour le rapport performance/sécurité.

Puis nous lançons le chiffrement du disque
```shell
cryptsetup --type luks2 --cipher aes-xts-plain64 --hash sha256 --iter-time 2000 --key-size 256 --pbkdf argon2i --sector-size 512 --use-urandom --verify-passphrase luksFormat /dev/sdb
```
|Commande|Description|
|:---:|:---:|
|Cryptsetup|Application permetant de chiffrer le disque|
|--type luks2|On utilise LUKS 2 pour effectuer le chiffrement|
|--cipher aes-xts-plain64|On utilise l'aes-xts comme algorithm de chiffrement|
|--hash sha256|Nous utiliserons le sha256 comme fonction de hachage|
|--iter-time 2000|C'est le temps en milliseconde consacré au déchiffrement de la passphrase|
|--key-size 256|C'est la taille de la clé, il est préférable de mettre la même taille que le chiffrement|
|--pbkdf argon2i|Nous choisirons l'argon2i pour le déchiffrement|
|--sector-size 512|Cet argument permet de définir la taille pris sur le disque pour stocker les données de l'encryption|
|--use-urandom|Nous utiliserons les nombres aléatoire stocké dans le urandom|
|--verify-passphrase luksFormat|Permet de check que la passphrase est bien au format LUKS|
|/dev/sdb|Défini le device à chiffrer|


Une fois chiffrer, on peux déchiffrer le disque avec la commande suivante
```shell
cryptsetup luksOpen /dev/sdb data
```

Nous pourrons ensuite créer un système de fichier 
```shell
mkfs.ext4 /dev/mapper/data
```

Nous pouvons vérifier les informations de chiffrement 
```shell
cryptsetup luksDump /dev/sdb
```

Puis monter le répertoire chiffrer que l'on a appelé **data**
```shell
umount /dev/mapper/data
```

Enfin, nous pouvons vérifier que le disque est bien monté et chiffré
```shell
lsblk
```
```shell
NAME              MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda                 8:0    0 136.1G  0 disk  
├─sda1              8:1    0   953M  0 part  /boot
└─sda2              8:2    0 135.2G  0 part  
  ├─vgroot-lvroot 254:0    0 111.8G  0 lvm   /
  ├─vgroot-lvswap 254:1    0   3.7G  0 lvm   [SWAP]
  ├─vgroot-lvtmp  254:2    0   9.3G  0 lvm   /tmp
  └─vgroot-lvhome 254:3    0  10.4G  0 lvm   /home
sdb                 8:16   0  14.6T  0 disk  
└─data            254:4    0  14.6T  0 crypt /data
```
*sdb TYPE **crypt***

## Tips

Il est aussi possible de refermer l'acces au dique chiffrer
```shell
cryptsetup luksClose /dev/sdb
```

Il est aussi possible d'ajouter plusieurs clé de déchiffrement (max 8)
```shell
cryptsetup luksAddKey /dev/sdb
```