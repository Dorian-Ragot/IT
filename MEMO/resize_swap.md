# Supprimer le swap et augmenter une autre partition

Cette note a pour objectif de permettre de supprimer/redimensionner la partition swap.
Elle comporte aussi une note pour agrandire la taille d'une autre partition.

## Supprimer/Redimensionner la partition swap

Premièrement, nous listons l'ensemble des volumes logiques pour réccupérer les informations.
```shell
lvscan
```
```shell
  ACTIVE            '/dev/vgroot/lvroot' [<111.76 GiB] inherit
  ACTIVE            '/dev/vgroot/lvswap' [3.72 GiB] inherit
  ACTIVE            '/dev/vgroot/lvtmp' [9.31 GiB] inherit
  ACTIVE            '/dev/vgroot/lvhome' [<10.40 GiB] inherit
```

Une fois que l'on a localiser le nom et la taille de la partition swap, nous pouvons stopper le swap.
*Attention, il est important que la charge en RAM ne soit pas a 100%*
```shell
swapoff -a
```

Ensuite nous pouvons désactiver la partition swap
```shell
lvchange /dev/vgroot/lvswap -a n
```
Et enfin nous pouvons la supprimer
```shell
lvremove /dev/vgroot/lvswap 
```
Nous pouvons vérifier la suppression du swap 
```shell
lvscan
```
```shell
  ACTIVE            '/dev/vgroot/lvroot' [<111.76 GiB] inherit
  ACTIVE            '/dev/vgroot/lvtmp' [9.31 GiB] inherit
  ACTIVE            '/dev/vgroot/lvhome' [<10.40 GiB] inherit
```

## Ajouter une partition swap

Il est aussi possible d'ajouter une partition swap.

Dans notre cas, permettre de réduire la partition swap qui étais à 4G au part avant.

On commence par récupérer les informations des groupes de volume
```shell
vgdisplay
```
```shell
  --- Volume group ---
  VG Name               vgroot
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                4
  Open LV               4
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               135.19 GiB
  PE Size               4.00 MiB
  Total PE              34609
  Alloc PE / Size       34609 / 135.19 GiB
  Free  PE / Size       0 / 0   
  VG UUID               #############################
```

Une fois que l'on a récupérer le nom du groupe de volume, on creer la partition swap de la taille que l'on souhaite (limitte de taille disponible).
Il est préférable de mettre le même nom que l'ancienne partition pour évité d'avoir a réécrire le point de montage swap dans /etc/fstab. 
```shell
lvcreate -L 2G -n lvswap vgroot
```
Ici, on créer un volume logique de 2G *(-L 2G)* que l'on nomme lvswap *(-n lvswap)*

Une fois notre partition swap créer, il faut la zone d'échange swap
```shell
mkswap /dev/vgroot/lvswap
```

Enfin, nous démarrons notre partition swap
```shell
swapon -a
```

Nous vérifion la disponiblilité du swap
```shell
free -h
```
```shell
               total        used        free      shared  buff/cache   available
Mem:            62Gi       721Mi       535Mi       4.0Mi        61Gi        61Gi
Swap:          1.7Gi       3.0Mi       1.7Gi

```

## Etendre un partition 

Note pour étendre simplement une partiton.

Ici, nous attribuons le reste de notre disque à la partition root
D'abord, nous listons la taille de disque disponible

```shell
df -h
```

Puis nous étendons la taille (en block)
```shell
lvresize -r -l +14975 /dev/vgroot/lvroot
```
Enfin, nous vérifions la taille de notre partition
```shell
df -h
```
