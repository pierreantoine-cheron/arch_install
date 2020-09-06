# arch_install
my script and procedure for installing arch

## Avant de commencer

J'ai téléchargé l'iso d'arch en torrent depuis le site officiel, que j'ai mise sur une clef usb avec [Ventoy](https://www.ventoy.net/en/index.html) installé dessus (si vous ne connaissez pas ventoy allez voir ça vaut le coup).

## Préparation du disque

J'ai partitionné mon disque (appelé "nvme0n1") comme ceci:
- /dev/nvme0n1p1 en fat32, partition pour le bootloader de 500 Mo; :heavy_exclamation_mark: l'étiqueter EF00 pour que ça fonctionne bien
- /dev/nvme0n1p2, partition de swap de 8 Go pour pouvoir mettre mon système en hibernation
- /dev/nvme0n1p3 en btrfs, partition pour le système de 50 Go
- /dev/nvme0n1p4 en btrfs, partition pour les utilisateurs de 90 Go
- /dev/nvme0n1p5 en ntfs, partition pour Windows déja installé de ~90 Go

Cela faisait un moment que je voulais essayer btrfs qui a maintenant l'air d'être mature comme le laisse notamment penser l'adoption par fedora ou [ce retour d'expérience](https://sebsauvage.net/wiki/doku.php?id=btrfs#apres_7_mois_sous_btrfs); je passe donc le pas.

## Installation

Booter sur le disque d'installation.

J'ai un clavier américain donc pas besoin de modifier la disposition par défaut.
Pour un clavier français, taper:
```sh
loadkeys fr
```

On monte ensuite les partitions du disque d'installation:
- la racine sur /mnt
- la partition d'efi sur /mnt/boot/efi
- la partition utilisateur sur /mnt/home
```sh
mount /dev/nvme0n1p3 /mnt
mkdir /mnt/{boot,boot/efi,home}
mount /dev/nvme0n1p1 /mnt/boot/efi
mount /dev/nvme0n1p4 /mnt/home
```

Pour accélerer la suite, on commence par installer [Reflector](https://wiki.archlinux.org/index.php/Reflector) qui permet d'obtenir le serveur le plus rapide pour tous les paquets qu'on va télécharger par la suite.
```sh
pacstrap /mnt reflector
reflector --latest 200 --sort rate --save /etc/pacman.d/mirrorlist
```

On installe tous les paquets dont ont va avoir besoin:
```sh
pacstrap /mnt base base-devel pacman-contrib grub os_prober efibootmgr
```
et d'autres utiles mais pas forcément obligatoires comme TLP pour l'autonomie des portables ou les microcodes de votre processeur, pour moi `intel_ucode`:
:heavy_exclamation_mark: l'utilisation de TLP avec btrfs nécéssite quelques précautions (dans la config de TLP, écrire `SATA_LINKPWR_ON_BAT=max_performance`)
```sh
pacstrap /mnt zip unzip p7zip vim mc alsa-utils syslog-ng mtools dosfstools lsb-release ntfs-3g exfat-utils bash-completion tlp
```




```sh

```
