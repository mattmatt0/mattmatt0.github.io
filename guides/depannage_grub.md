# Guide de dépannage de GRUB
Si vous avez essayé d'installer Linux sur un ordinateur relativement récent, il se peut que vous ayez eu l'erreur suivante:

Ou en fait une erreur par rapport à l'installation de GRUB. C'est une erreur fréquente sur les ordinateurs en UEFI (j'expliquerai de quoi il s'agit juste après). Malheureusement, la solution à ce problème n'est pas facile à trouver, trop peu de documentations l'évoquent. Dans ce guide, je vais donc vous expliquer comment réparer une installation Linux après une erreur d'installation de GRUB.
Ce guide est à suivre après l'installation du système. Le mieux est de lancer l'installation, d'attendre l'erreur et suivre ce guide, comme ça vous aurez une installation propre. C'est pour ça que je donne l'instruction suivante: **N'éteignez pas votre ordinateur après avoir vu l'erreur d'installation, on va se servir du système en cours de fonctionnement pour dépanner l'installation!**
## Quelques explications sur le problème
Quand vous installez un ou plusieurs systèmes d'exploitation sur une machine, un programme, du nom de bootloader, va être installé avec votre (ou vos) système. Le rôle de ce programme est de trouver le système d'exploitation, de le charger avec les fichiers, les paramètres nécessaires à son bon fonctionnement. GRUB (**GR**and **U**nified **B**ootloader) est un programme qui permet de faire ça.
L'erreur que vous rencontrez signifie que ce programme n'a pas pu être correctement installé. En revanche, votre système, lui, a bien été installé, et est complètement fonctionnel. Mais comme il n'y a pas de programme pour le démarrer, bah... il démarre pas 😀. Mais la bonne nouvelle, c'est qu'on peut le réparer 🥳!
Mais qu'est ce qui cause l'erreur? Pour avoir mené mon enquête, il semblerait que ce soit dû à un problème de variables UEFI.

## UEFI?
Dans votre ordinateur, il existe un programme qui permet de gérer les composants de votre ordinateur. C'est lui qui va démarrer les disques et démarrer le bootloader notamment. Ce programme s'appelle BIOS (**B**asic **I**nput/**O**utput **S**ystem). Récemment<sup>il y a moins de 10 ans<sup>, un nouveau type de BIOS apparaît: l'UEFI. Il permet d'avoir une interface graphique avec un souris, un démarrage plus rapide... Mais il cause parfois quelques problèmes!
Ce qu'il se passe en fait, c'est que lors de l'installation de GRUB, le programme qui installe GRUB va essayer de modifier des données de l'UEFI pour le configurer pour GRUB. Mais l'UEFI ne laisse pas le programme le faire, ce qui cause une erreur. Et GRUB n'est pas installé.

## Comment on dépanne?
Je vais donner les étapes que l'on va suivre dans ce guide:
1 - S'assurer que l'on est bien en UEFI
2 - Trouver les partitions systèmes et ESP
3 - Monter ces partitions
4 - Réinstaller GRUB
5 - Configurer GRUB

## Savoir si on est en UEFI ou en Legacy
Avant tout, ce guide tente de trouver une solution pour certains problèmes liés à l'UEFI. Il ne sert à rien de le suivre si vous avez un ordinateur en Legacy (pas UEFI quoi). On veut donc vérifier si notre ordinateur est en UEFI. Pour ce faire, ouvrez un terminal: cherchez terminal (ou konsole) si vous avez un endroit où chercher des applications, si vous avez un menu d'applications allez dans applications -> Outils Système (ou Système, ça dépend) -> Terminal (ou konsole).
Ensuite, écrivez dans ce terminal ```bash
ls /sys/firmware/efi```
Si vous avez une erreur comme quoi ce dossier n'existe pas, c'est que vous n'êtes pas en UEFI. Si vous avez des trucs comme config_table, efivars, runtime... C'est bon!

## Trouver les partitions systèmes et ESP
Les opérations qui vont suivre nécessitent les permissions du superutilisateur ("root"), une sorte d'équivalent encore plus puissant à l'administrateur Windows. Pour les avoir, vous pouvez écrire ```sudo ```  avant chacune des commandes (n'oubliez pas de mettre un espace entre sudo et la commande que vous voulez éxécuter) qui va suivre, ou alors écrire ```sudo su``` pour vous identifier en tant que root (et donc éxécuter chacune des commandes en tant que root).
