# DISK RESIZE

## Augmenter la taille d'un volume LVM

### Redimentionner du LV
La commande peut être utilisé sur un disque monté.

```shell
# lvextend -L +10G /dev/vg_foobar/var
```

## Augmenter la taille d'un volume physique
Intervient si le disque monté n'a pas la totalité de son espace de configuré. Le but est donc de réécrire la table de partition sans perte de donnée.
La perte de donnée peut intervenir si les partitions sont consécutive et que l'on augmente la première ce qui supprime celle d'après.

Premièrement, il faut démonter la partition pour pouvoir interagir dessus

```shell
# unmount /dev/<partition name>
```
Ensuite, on regarde la taille de disque libre à attribuer à la partition

```shell
# parted /dev/<partition name> print free
```

On supprime la partition pour la recréer.

Il n'y a pas de perte de donnée si l'on respecte la structure du disque (le START et le END)

```shell
# parted /dev/<partition name>
```

On liste les partitions pour avoir l'ID de la partition à supprimer

```shell
(parted) # print
```

On peut ensuite supprimer la partition

```shell
(parted) # rm <id>
```

Puis la recréer 

```shell
(parted) # mkpart primary [fs-type] <START value> <END value with add size>
```

Enfin quitter parted

```shell
(parted) # quit
```

## Exemple pour une partition avec 20Go de plus
>START 50kB - END 100GB

```shell
# rm 1
```
```shell
# mkpart primary [fs-type] 50kB 120Go
```
```shell
# quit
```