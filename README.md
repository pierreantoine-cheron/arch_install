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
```bash
loadkeys fr
```
On vérifie que le mode de boot est bien UEFI:
```bash
ls /sys/firmware/efi/efivars
```
Si le dossier n'exite pas, le système doit etre booté en BIOS.

On monte ensuite les partitions du disque d'installation:
- la racine sur /mnt
- la partition d'efi sur /mnt/efi
- la partition utilisateur sur /mnt/home
```bash
mount /dev/nvme0n1p3 /mnt
mkdir /mnt/{efi,home}
mount /dev/nvme0n1p1 /mnt/efi
mount /dev/nvme0n1p4 /mnt/home
```

La suite nécessite une connexion internet je suis connecté en ethernet via RJ45, si vous êtes en Wi-Fi, je vous invite à suivre la section "connect to the internet" du [guide d'installation d'arch linux](https://wiki.archlinux.org/index.php/Installation_guide)

Pour accélerer la suite, on commence par installer [Reflector](https://wiki.archlinux.org/index.php/Reflector) qui permet d'obtenir le serveur le plus rapide pour tous les paquets qu'on va télécharger par la suite.
```bash
pacstrap /mnt reflector
reflector --latest 200 --sort rate --save /etc/pacman.d/mirrorlist
```

On installe tous les paquets dont ont va avoir besoin dont les microcodes de votre processeur, pour moi `intel_ucode`:
```bash
pacstrap /mnt base base-devel intel-ucode linux linux-firmware pacman-contrib grub os_prober efibootmgr
```
et d'autres utiles mais pas forcément obligatoires comme TLP pour l'autonomie des portables:
:heavy_exclamation_mark: l'utilisation de TLP avec btrfs nécéssite quelques précautions (dans la config de TLP, écrire `SATA_LINKPWR_ON_BAT=max_performance`)
```bash
pacstrap /mnt zip unzip p7zip vim mc alsa-utils syslog-ng mtools dosfstools lsb-release ntfs-3g exfat-utils bash-completion tlp btrfs-prog
```

On génère maintenant la table de partition:
```bash
genfstab -U -p /mnt >> /mnt/etc/fstab
```
Enfin on entre dans notre installation toute fraiche pour passer à la suite:

```bash
arch-chroot /mnt
```

## Configuration

### Localisation
Pour changer le clavier, editer /etc/vconsole.conf :

```bash
KEYMAP=fr-latin9
```
et pour la police (important pour bien afficher les apostrophes par exemple), toujours dans `/etc/vconsole.conf` :
```bash
FONT=eurlatgr
```

Pour la localisation française on édite `/etc/locale.conf`:
```bash
LANG=fr_FR.UTF-8
```
et on décommente les lignes
```bash
en_US.UTF-8 UTF-8
fr_FR.UTF-8 UTF-8
```
dans `/etc/locale.gen`.
Enfin on génère la localisation:
```bash
locale-gen
```
On exporte la langue pour qu'elle soit prise en compte dans la session courante:
```bash
export LANG=fr_FR.UTF-8
```

### Nommage du pc
```bash
vim /etc/hostname
```
on écrit dans le fichier le nom qu'on veut donner à la machine.

### Date et heure
Je choisis le fuseau horaire de Paris:
```bash
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
```
et l'heure UTC (il faudra aussi activer l'UTC dans windows):
```bash
hwclock --systohc --utc
```

### Installation de Grub

Grub est le chargeur d'amorçage que j'ai choisi, il permettra de choisir entre windows et arch linux au démarrage de mon PC.
```bash
mount | grep efivars &> /dev/null || mount -t efivarfs efivarfs /sys/firmware/efi/efivars
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=arch_grub --recheck
mkdir /efi/EFI/boot
cp /efi/EFI/arch_grub/grubx64.efi /efi/EFI/boot/bootx64.efi
grub-mkconfig -o /efi/grub/grub.cfg
```

### Finalisation de l'installation de base

On active le swap:
```bash
swapon /dev/nvme0n1p2
```

On choisis le mot de passe de root:
```bash
passwd root
```

On installe un gestionnaire de réseau:
```bash
pacman -Syy networkmanager
systemctl enable NetworkManager
```

Certains paquets ne sont disponibles qu'en 32 bits, pour pouvoir les installer on édite `/etc/pacman.conf` et sous la section `[multilib]` on décommente la ligne
```bash
#Include = /etc/pacman.d/mirrorlist
```

On peut maintenant quitter l'installation pour retourner sur le disque d'installation:
```bash
exit
```
On démonte proprement les partitions:
```bash
umount -R /mnt
```

Enfin on peut éteindre la machine:
```bash
shutdown now
```

Il ne reste plus qu'à retirer le périphérique d'installation.

# To Do
zsh
i8kutils dell fan control
dans la config de TLP, écrire `SATA_LINKPWR_ON_BAT=max_performance`
activer `fstrim.service` dans systemd du paquet util-linux, ce service trim le ssd toutes les semaines une opération qui permet de conserver les performances du SSD
pour verifier que l'opération est supportée par le SSD `lsblk --discard` disc-gran & disc max !=0 signifie support
fingerprint-gui AUR
libinput touchpad
CUPS impression

utc time dans windows
```bash

```
