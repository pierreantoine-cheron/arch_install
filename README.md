# arch_install
my script and procedure for installing arch

## Avant de commencer

J'ai téléchargé l'iso d'arch en torrent depuis le site officiel, que j'ai mise sur une clef usb avec [Ventoy](https://www.ventoy.net/en/index.html) installé dessus (si vous ne connaissez pas ventoy allez voir ça vaut le coup).

## Préparation du disque

J'ai partitionné mon disque (appelé "nvme0n1") comme ceci:
- /dev/nvme0n1p1 en fat32, partition pour le bootloader de 500 Mo; :heavy_exclamation_mark: étiqueter EF00 pour que ça fonctionne bien
- /dev/nvme0n1p2, partition de swap de 8 Go pour pouvoir mettre mon système en hibernation
- /dev/nvme0n1p3 en btrfs, partition pour le système de 50 Go
- /dev/nvme0n1p4 en btrfs, partition pour les utilisateurs de 90 Go
- /dev/nvme0n1p5 en ntfs, partition pour Windows déja installé de ~90 Go

Cela faisait un moment que je voulais essayer btrfs qui a maintenant l'air d'être mature comme le laisse notamment penser l'adoption par fedora ou [ce retour d'expérience](https://sebsauvage.net/wiki/doku.php?id=btrfs#apres_7_mois_sous_btrfs); je passe donc le pas.

