---
layout: home

hero:
  name: "Archlinux"
  text: "Installation Guide"
  tagline: "A Reference for Repeatability"
  actions:
    - theme: alt
      text: Archwiki
      link: https://TODO

features:
  - title: Bootloader
    details: Systemd-boot will be used as the bootloader
  - title: Display Server
    details: Wayland along with sway window manager will be used
  - title: Encryption
    details: LUKS will be used to encrypt the root and home partitions
---

### Drive Preparation
After booting into the installation media, securely wipe the disk
```
wipefs -a /dev/nvme0n1
nvme format /dev/nvme0 -n 1 -s 1
```
::: danger CAUTION
ALL data will be ERASED irrecoverably ! Proceed with caution !
:::

### Drive Partitioning
Three partitions are required:<br>
Root (/), Boot (/boot), Home (/home)
```
gdisk /dev/nvme0n1 ► n ► Size: 1G   ► Type: ef00 (/boot)
                   ► n ► Size: 60G  ► Type: 8309 (/root)
                   ► n ► Size: REST ► Type: 8309 (/home)
                   ► w ► y
```

### Setup LUKS containers
```
cryptsetup luksFormat /dev/nvme0n1p2
cryptsetup luksFormat /dev/nvme0n1p3

cryptsetup luksOpen /dev/nvme0n1p2 luksRoot
cryptsetup luksOpen /dev/nvme0n1p3 luksHome
```

### Make Filesystems
Boot (/boot) partition will use fat32<br>
Root (/) and Home (/home) partitions will use ext4
```
mkfs.fat -F32 -n EFI /dev/nvme0n1p1
mkfs.ext4 -L root /dev/mapper/luksRoot
mkfs.ext4 -L home /dev/mapper/luksHome
```

### Mount Filesystems
```
mount /dev/mapper/luksRoot /mnt

mkdir /mnt/home
mkdir /mnt/boot
mkdir /mnt/etc

mount /dev/mapper/luksHome /mnt/home
mount /dev/nvme0n1p1 /mnt/boot
```

### Check internet & Set date/time
```
curl icanhazip.com

timedatectl set-timezone Asia/Kolkata
timedatectl set-ntp true
```

### Generate Pacman Mirrorlist
```
reflector -p https -l 200 -n 20 --sort rate --save /etc/pacman.d/mirrorlist
pacman -Sy vim
```

### Generate fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
```

### Generate crypttab
```
vim /mnt/etc/crypttab
[crypttab contents]
# <name>  <device>                   <password>  <options>
luksHome  UUID=uuid-of-block-device  none        luks
```

### Chroot into the installation
```
pacstrap -i /mnt base
arch-chroot /mnt
```

### Install the necessary packages
```
curl https://TODO > packages.sh
chmod +x packages.sh
./packages.sh
```
::: details Packages
```
linux linux-headers base-devel openssh linux-firmware vim git networkmanager netctl dialog reflector amd-ucode sbctl efibootmgr dosfstools mtools mesa pwgen libva-mesa-driver wayland sway swaylock swayidle swaybg fuzzel waybar gvfs alacritty thunar fragments celluloid gvfs-smb pipewire pipewire-pulse gvfs-mtp gsettings-desktop-schemas pamixer pavucontrol otf-font-awesome grim slurp swappy rsync gedit ristretto corectrl qt5-wayland qt6-wayland wget android-tools exfatprogs zathura zathura-pdf-mupdf usbutils htop neofetch keepassxc tumbler thunar-archive-plugin thunar-volman file-roller ttf-dejavu gtk-engine-murrine noto-fonts-cjk noto-fonts-emoji noto-fonts code ttf-hack-nerd gnome-theme-extra ttf-sourcecodepro-nerd ttf-ubuntu-font-family github-cli papirus-icon-theme
```
:::

### Enable necessary services
```
systemctl enable sshd
systemctl enable NetworkManager
```

### Switch to systemd based initramfs
Edit the HOOKS line in mkinitcpio.conf
```
vim /etc/mkinitcpio.conf
HOOKS=(systemd keyboard autodetect microcode modconf kms sd-vconsole block sd-encrypt filesystems fsck)
mkinitcpio -P
```

### Set the locale
Uncomment the line with the correct locale<br>
'en_US.UTF-8 UTF-8'
```
vim /etc/locale.gen
locale-gen
```

### Setup user accounts
Setup the Root Password
```
passwd
```

Create a new user with sudo priviledges
```
useradd -m -g users -G wheel username
passwd username
```

Uncomment the 'wheel' line
```
EDITOR=vim visudo
```

### Setup the bootloader
```
bootctl --esp-path=/boot install

vim /boot/loader/loader.conf
[loader.conf contents]
default archlinux.conf
timeout 0
console-mode max

vim /boot/loader/entries/archlinux.conf
[archlinux.conf contents]
title  Archlinux
linux  /vmlinuz-linux
initrd /amd-ucode.img
initrd /initramfs-linux.img
options rd.luks.name=uuid-of-block-device=luksRoot root=/dev/mapper/luksRoot rd.luks.options=timeout=30s,password-echo=no,tries=3 rw quiet
```
` Exit the chroot environment, unplug the installation media & reboot`
::: tip Note
Install the `os-prober` package only if you want other Operating Systems to be recognized by the bootloader
:::

### Configure date & time settings
```
timedatectl set-timezone Asia/Kolkata
systemctl enable systemd-timesyncd
timedatectl set-ntp true
hwclock --systohc
```

### Configure the hostname
```
hostnamectl set-hostname archlinux

vim /etc/hosts
[/etc/hosts contents]
127.0.0.1 localhost
127.0.1.1 archlinux
```

### Setup Secureboot
Reboot into UEFI & set a strong password<br>
Clear the factory keys from UEFI<br>
Save & Reboot

```
sbctl create-keys
sbctl enroll-keys --tpm-eventlog
sbctl sign -s /boot/EFI/BOOT/BOOTX64.EFI
sbctl sign -s /boot/EFI/systemd/systemd-bootx64.efi
sbctl sign -s /boot/vmlinuz-linux
sbctl verify
```

Reboot into UEFI & enable Secureboot<br>
Save and Reboot<br>

### Setup the Graphical User Interface
```
git clone https://github.com/TODO
cd swaywm
./setup.sh
reboot
```
`Now, hopefully if you've made no mistakes then you should be greeted with the login screen !`

<br>
<h1 :style="{ textAlign: 'center' }">
Now you can say "I use Arch, By the Way ¬‿¬
</h1>
