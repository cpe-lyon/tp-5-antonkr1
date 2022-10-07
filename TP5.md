
# TP5 Systèmes de fichiers, partitions et disques

# _Exercice 1. Disques et partitions_

1 - Je me rend dans les paramètres de vsphère pour y ajouter un disque dur en passant par "actions > modifier les paramètres > ajouter un périphérique > Disque Dur. Je créer donc un disque dur de 5Go en dynamiquement alloués. (Cela permet de ne pas directement prendre 5Go de stockage à la création du disque, il prendra uniquement 1Go si il ne stock que 1Go de donnés par exemple)  

2 - Je m'assure ensuite que mon disque dur est bien présent sur ma VM en utilisant "lsblk"  

3 - On va ensuite partitionner le nouveau disque sdb en 2 partitions différentes avec "sudo fdisk /dev/sdb" :  
- La première  : 
![194076349-551e8f95-68f2-442c-bffc-b1e00091af33](https://user-images.githubusercontent.com/113091817/194652097-1348c6c1-7437-46aa-97d7-c093837c2304.png) 
- La deuxième :   
![194078017-9e830773-f1ad-4aaa-aca8-98138d44ea60](https://user-images.githubusercontent.com/113091817/194652397-42f331f5-68ec-4c47-abab-3a8ca04d55b0.png)  
Une fois nos deux partitions linux (2Go) et NTFS (3Go) on sauvegarde avec w.  

4 - Je formate mes deux partitions en utilisant la commande "mkfs"
```
sudo mkfs -t ntfs /dev/sdb5
```
5 - La commande "df -T" ne fonctionne pas sur notre nouveau disque étant donné qu'il n'est pas encore monté.  

6 - Pour se faire il faut se rendre dans le fichier "/etc/fstab" qui est le fichier qui liste les partitions qui sont automatiquement montées au démarrage. Il faut commencer par créer des points de montage : 
```
sudo mkdir /media/data
sudo mkdir /media/win
```
Puis ensuite modifier le ficher "/etc/fstab" en y ajoutant :
```
/dev/sdb1 /media/data ext4 defaults 0 2
/dev/sdb2 /media/win ntfs defaults 0 2
```

7 - Je fais un "sudo mount -a" avant de redémarrer ma machine.  

8 - Je n'ai pas accès à la fonctionnalité permettant de la faire (Virtual remote console).  
9 -   

# _Exercice 2. Partitionnement LVM_

1 - Pour démonter mes deux systèmes de fichier je fais :
```
sudo umount /media/data
sudo umount /media/win
```
Je retourne ensuite de le fichier "/etc/fstab" pour y retirer ce qu'on avait ajouté précédement.  

2 - Les deux partitions étant supprimées, j'en créer une nouvelle en LVM : 

![194238424-2eb357ce-9a85-431c-844c-d4e1a49006e1](https://user-images.githubusercontent.com/113091817/194660806-d5083529-6c9b-4c90-b0ab-a17d594cd6ee.png)  
3 - 
```
sudo pvcreate /dev/sdb1
sudo pvdisplay
```
![Exo 2 q3](https://user-images.githubusercontent.com/113091817/194661937-e6733e13-0753-4c0f-91f7-5b1258771b2a.png)    

4 - Je fais la même chose que la question précédente mais avec "vgcreate" et "vgdisplay"
```
sudo vgcreate /dev/sdb1
sudo vgdisplay
```
5 - Pour créer notre volume logique lvData occupant l'intégralité de l'espace disponible du disque on utilise :
```
sudo lvcreate -n lvData -l 100%FREE vg1
```

6 - On va dans un premier temps créer la partition grâce à un :
```
sudo fdisk /dev/vg1/lvData
```
Puis la formater : 
```
mkfs.ext4 /dev/bg1/lvData
```
Et ensuite pour finir on retourne dans le fichier "/etc/fstab" pour y ajouter notre nouvelle partition comme vu précédemment.  

7 - Il suffit de suivre les mêmes procédures que la question 1 de l'exercice 1, puis de la question 2 et 3 de l'exercice 2. On vérifie bien avec un "lsblk" que notre nouveau disque est bien présent.

8 - Pour ajouter le nouveau disque au groupe de volumes : 
```
sudo vgextend sysbg /dev/sdc1
```
9 - La commande permettant d'agrandir le volume logique : 
```
sudo lvresize -l +100%FREE /dev/vg01/lvData
sudo resize2fs /dev/vg01/lvData
```

