# Supprimer le swap et augmenter une autre partition
lvscan
lvremove /dev/integration-mcs-vg/swap_1 
swapoff --help
swapoff -a
lvchange /dev/integration-mcs-vg/swap_1 -a n
lvremove /dev/integration-mcs-vg/swap_1 
vgdisplay
lvcreate -L 2G -n swap_1 integration-mcs-vg
mkswap --help
mkswap /dev/integration-mcs-vg/swap_1
swapon -a
free -h
lvresize -r -l +14975 /dev/integration-mcs-vg/root
df -h
