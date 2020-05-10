# Guide de dépannage de GRUB
Si vous avez essayé d'installer Linux sur un ordinateur relativement récent, il se peut que vous ayez eu l'erreur suivante:

Ou en fait une erreur par rapport à l'installation de GRUB. C'est une erreur fréquente sur les ordinateurs en UEFI (j'expliquerai de quoi il s'agit juste après). Malheureusement, la solution à ce problème n'est pas facile à trouver, trop peu de documentations l'évoquent. Dans ce guide, je vais donc vous expliquer comment réparer une installation Linux après une erreur d'installation de GRUB.
Ce guide est à suivre après l'installation du système. Le mieux est de lancer l'installation, d'attendre l'erreur et suivre ce guide, comme ça vous aurez une installation propre. C'est pour ça que je donne l'instruction suivante: **N'éteignez pas votre ordinateur après avoir vu l'erreur d'installation, on va se servir du système en cours de fonctionnement pour dépanner l'installation!**
## Quelques explications sur le problème
Quand vous installez un ou plusieurs systèmes d'exploitation sur une machine, un programme, du nom de bootloader, va être installé avec votre (ou vos) système. Le rôle de ce programme est de trouver le système d'exploitation, de le charger avec les fichiers, les paramètres nécessaires à son bon fonctionnement. GRUB (**GR**and **U**nified **B**ootloader) est un programme qui permet de faire ça.
L'erreur que vous rencontrez signifie que ce programme n'a pas pu être correctement installé. En revanche, votre système, lui, a bien été installé, et est complètement fonctionnel. Mais comme il n'y a pas de programme pour le démarrer, bah... il démarre pas 😀. Mais la bonne nouvelle, c'est qu'on peut le réparer 🥳!
Mais qu'est ce qui cause l'erreur? Pour avoir mené mon enquête, il semblerait que ce soit dû à un problème de variables UEFI.

## UEFI?
Dans votre ordinateur, il existe un programme qui permet de gérer les composants de votre ordinateur. C'est lui qui va démarrer les disques et démarrer le bootloader notamment. Ce programme s'appelle BIOS (**B**asic **I**nput/**O**utput **S**ystem). Récemment<sup>il y a moins de 10 ans</sup>, un nouveau type de BIOS apparaît: l'UEFI. Il permet d'avoir une interface graphique avec un souris, un démarrage plus rapide... Mais il cause parfois quelques problèmes!
Ce qu'il se passe en fait, c'est que lors de l'installation de GRUB, le programme qui installe GRUB va essayer de modifier des données de l'UEFI pour le configurer pour GRUB. Mais l'UEFI ne laisse pas le programme le faire, ce qui cause une erreur. Et GRUB n'est pas installé.

## Comment on dépanne?
Je vais donner les étapes que l'on va suivre dans ce guide:
1. S'assurer que l'on est bien en UEFI
2. Trouver les partitions systèmes et ESP
3. Monter ces partitions
4. Réinstaller GRUB
5. Configurer GRUB

## Savoir si on est en UEFI ou en Legacy
Avant tout, ce guide tente de trouver une solution pour certains problèmes liés à l'UEFI. Il ne sert à rien de le suivre si vous avez un ordinateur en Legacy (pas UEFI quoi). On veut donc vérifier si notre ordinateur est en UEFI. Pour ce faire, ouvrez un terminal: cherchez terminal (ou konsole) si vous avez un endroit où chercher des applications, si vous avez un menu d'applications allez dans applications -> Outils Système (ou Système, ça dépend) -> Terminal (ou konsole).
Ensuite, écrivez dans ce terminal 
```bash
ls /sys/firmware/efi
```
Si vous avez une erreur comme quoi ce dossier n'existe pas, c'est que vous n'êtes pas en UEFI. Si vous avez des trucs comme config_table, efivars, runtime... C'est bon!

## Trouver les partitions systèmes et ESP
Les opérations qui vont suivre nécessitent les permissions du superutilisateur ("root"), une sorte d'équivalent encore plus puissant à l'administrateur Windows. Pour les avoir, vous pouvez écrire `sudo ` avant chacune des commandes (n'oubliez pas de mettre un espace entre sudo et la commande que vous voulez éxécuter) qui va suivre, ou alors écrire `sudo su ` pour vous identifier en tant que root (et donc éxécuter chacune des commandes en tant que root).<br/>
Vos disques (j'entends par là tout ce qui sert à contenir des données) sont coupés en plusieurs parties, appelées partitions. Par exemple, il se peut que sur une machine Windows il y ait deux partitions: C: et D: (eh oui, les deux sont sur le même disque). Il y a plusieurs types de partitions, car il y a plusieurs façon de stocker des données sur un disque. On dit qu'une partition est formatée à un format quand cette partition utilise ce format (cette méthode de stockage) de données.<br/>
Pour repérer les partitions, commencez par les lister, avec
```bash
fdisk -l
```
(en tant que root, ne l'oubliez pas!)
Il devrait vous affichier quelque chose comme ça:
```
Disque /dev/sda : 119,25 GiB, 128035676160 octets, 250069680 secteurs
Modèle de disque : SAMSUNG MZNLN128
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 4096 octets
taille d'E/S (minimale / optimale) : 4096 octets / 4096 octets
Type d'étiquette de disque : gpt
Identifiant de disque : DE1CB64F-31E9-4DEE-B809-3D3FD5A49539

Périphérique     Début       Fin  Secteurs Taille Type
/dev/sda1         2048  83888127  83886080    40G Système de fichiers Linux
/dev/sda2     83888128  84150271    262144   128M Système EFI
/dev/sda3     84150272 241436671 157286400    75G Système de fichiers Linux
/dev/sda4    241436672 250069646   8632975   4,1G Partition d'échange Linux


Disque /dev/loop0 : 93,94 MiB, 98484224 octets, 192352 secteurs
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 512 octets
taille d'E/S (minimale / optimale) : 512 octets / 512 octets


Disque /dev/loop1 : 171,101 MiB, 180338688 octets, 352224 secteurs
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 512 octets
taille d'E/S (minimale / optimale) : 512 octets / 512 octets


Disque /dev/loop2 : 373,8 MiB, 391933952 octets, 765496 secteurs
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 512 octets
taille d'E/S (minimale / optimale) : 512 octets / 512 octets


Disque /dev/loop3 : 93,8 MiB, 98336768 octets, 192064 secteurs
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 512 octets
taille d'E/S (minimale / optimale) : 512 octets / 512 octets


Disque /dev/loop4 : 174,77 MiB, 183242752 octets, 357896 secteurs
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 512 octets
taille d'E/S (minimale / optimale) : 512 octets / 512 octets


Disque /dev/loop5 : 310,82 MiB, 325902336 octets, 636528 secteurs
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 512 octets
taille d'E/S (minimale / optimale) : 512 octets / 512 octets
```
ou en anglais
```
Disk /dev/sda: 119.25 GiB, 128035676160 bytes, 250069680 sectors
Disk model: SAMSUNG MZNLN128
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: DE1CB64F-31E9-4DEE-B809-3D3FD5A49539

Device         Start       End   Sectors  Size Type
/dev/sda1       2048  83888127  83886080   40G Linux filesystem
/dev/sda2   83888128  84150271    262144  128M EFI System
/dev/sda3   84150272 241436671 157286400   75G Linux filesystem
/dev/sda4  241436672 250069646   8632975  4.1G Linux swap


Disk /dev/loop0: 93.94 MiB, 98484224 bytes, 192352 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop1: 171.101 MiB, 180338688 bytes, 352224 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop2: 373.8 MiB, 391933952 bytes, 765496 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop3: 93.8 MiB, 98336768 bytes, 192064 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop4: 174.77 MiB, 183242752 bytes, 357896 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop5: 310.82 MiB, 325902336 bytes, 636528 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```
Cette commande présente tous les disques avec les partitions qu'ils contiennent. Vous pouvez ignorer tous ceux qui commencent par /dev/loop
Intéressons-nous plutôt à `/dev/sda`:
```
Disque /dev/sda : 119,25 GiB, 128035676160 octets, 250069680 secteurs
Modèle de disque : SAMSUNG MZNLN128
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 4096 octets
taille d'E/S (minimale / optimale) : 4096 octets / 4096 octets
Type d'étiquette de disque : gpt
Identifiant de disque : DE1CB64F-31E9-4DEE-B809-3D3FD5A49539

Périphérique     Début       Fin  Secteurs Taille Type
/dev/sda1         2048  83888127  83886080    40G Système de fichiers Linux
/dev/sda2     83888128  84150271    262144   128M Système EFI
/dev/sda3     84150272 241436671 157286400    75G Système de fichiers Linux
/dev/sda4    241436672 250069646   8632975   4,1G Partition d'échange Linux
```
Il se peut que vous ayez d'autres disques, `/dev/sdb` par exemple, ou un truc comme `/dev/nvme....`
Intéressez-vous à chacun d'entre eux, on va essayer de trouver quel disque contient votre système.
Ici, /dev/sda fait 119,25 GiB, ce qui correspond à peu près à la taille du disque de mon ordinateur, 128 GiB. En revanche, si vous avez une taille d'environ 8 GiB, 16 GiB, 32 GiB, c'est plutôt votre clé USB/ CD live sur lequel vous avez démarré.
Une autre indication peut être le nom du disque: ici, c'est `Modèle de disque : SAMSUNG MZNLN128` qui indique le nom du notre disque, SAMSUNG MZNLN128. Si vous avez un nom qui rapelle celui de votre clé USB, c'est certainement votre clé USB.
Et si vous ne savez toujours pas, regardez les partitions à l'intérieur. Dans le mien, il y en a 4:
```
Périphérique     Début       Fin  Secteurs Taille Type
/dev/sda1         2048  83888127  83886080    40G Système de fichiers Linux
/dev/sda2     83888128  84150271    262144   128M Système EFI
/dev/sda3     84150272 241436671 157286400    75G Système de fichiers Linux
/dev/sda4    241436672 250069646   8632975   4,1G Partition d'échange Linux
```
Certaines sont des systèmes de fichiers Linux, on peut supposer que notre système d'exploitation Linux a été installé sur ces partitions (vous pouvez en avoir moins, mais en général si ça contient "système de fichiers Linux", c'est que c'est le disque où vous avez installé Linux).<br/>
Vous remarquerez que toutes les partitions commencent par `/dev/sda` (le nom de mon disque), suivi par leur numéro (1,2,3,4). En général, le système va être installé sur la première partition, `/dev/sda1` dans mon cas. 
