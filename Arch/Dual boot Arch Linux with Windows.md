#### Created by tilas01[www.github.com/tilas01] on GitHub
Do not remove my credit if sharing, modifying or publishing anything with this guide please.

With the above aside lets get started!
I hope this or any of my other guides can help you!

# Dual boot Arch Linux with Windows 10/11 (UEFI)

### Windows Setup

1. Ensure Secure Boot and Fast Startup are disabled

2. Open "Disk Management"

3. Right click on your C:\ Drive and select "Shrink Volume"

4. Shrink the volume the size you want your Arch install to be

---

### Arch Install

Flash to USB:

```
Follow this guide:
https://wiki.archlinux.org/title/USB_flash_installation_medium
```

Assumptions

```
This guide assumes:
/dev/sda is your disk
/dev/sda1 is your EFI Partition
/dev/sda2 is your Linux Swap Partition
/dev/sda3 is your Linux Filesystem Partition

These should be replaced in this guide where applicable
```

Verify the boot mode

```
# ls /sys/firmware/efi/efivars
```

Connect to Wi-Fi (Optional)

```
Follow this guide:
https://wiki.archlinux.org/title/Iwd#iwctl
```

Check your connection

```
# ip link
# ping archlinux.org
```

Enable SSH (Optional)

```
# passwd
# systemctl enable sshd

Find your machines local ip:
# ip a
```

Update the system clock

```
# timedatectl set-ntp true
```

Discover your disk

```
# fdisk -l
```

Partition the disk

```
# cfdisk /dev/sda

Create a 4GB "Linux Swap" Partition then a "Linux Filesystem" Partition using the rest of your free space.

Partiton Table:
--gpt--
(windows partitions)
/dev/sda2 4GB Linux Swap Partition
/dev/sda3 Linux Filesystem (rest of storage)

# fdisk -l (confirm partitioning succeeded)

Note: Feel free to use any partitioning tool you like.
```

Format the Partitions

```
# mkfs.ext4 /dev/sda3
# mkswap /dev/sda2
```

Mount the System

```
# mount /dev/sda3 /mnt
# mkdir /mnt/efi
# mount /dev/sda1 /mnt/efi
# swapon /dev/sda2
```

Setup your Mirrors (These will be copied to the Arch Install via pacstrap)

```
# pacman -Sy reflector

This command will find all https mirrors for your country, sort them by speed and save it.
# reflector --verbose -c {your_country} -p https --sort rate --save /etc/pacman.d/mirrorlist
```

Install Arch + Some Required Additional Packages

```
If you wish to use a kernel other than "linux" such as "linux-zen" replace the following:
"linux" with "linux-zen"
"linux-headers" with "linux-zen-headers"
Note: If you are not installing zen swap "zen" with "lts" or "hardened"
List of officially supported Kernels: https://wiki.archlinux.org/title/kernel

Depending on your processor add "intel-ucode" or "amd-ucode" to the following command, or if installing to a VM add neither.

# pacstrap /mnt base linux linux-firmware base-devel linux-headers nano git networkmanager openssh
```

Generate your fstab file

```
# genfstab -U /mnt >> /mnt/etc/fstab
```

Chroot into the system

```
# arch-chroot /mnt
```

Set the time zone

```
View available timezones:
# ls /usr/share/zoneinfo

Set timezone:
# ln -sf /usr/share/zoneinfo/{your_region}/{your_city} /etc/localtime
# hwclock --systohc
```

Set Locale

```
# nano /etc/locale.gen (uncomment en_US.UTF-8 UTF-8)
# locale-gen
# echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```

Set hostname

```
# echo "yourhostname" >> /etc/hostname
```

Add hostname to /etc/hosts

```
# nano /etc/hosts

Then add:
127.0.0.1        localhost
::1              localhost
127.0.1.1        {yourhostname}
```

Enable SSH (Optional)

```
# systemctl enable sshd
```

Enable NetworkManager

```
# systemctl enable NetworkManager
```

Install some nice to have packages (Optional)

```
# pacman -Sy vim neovim htop neofetch net-tools
```

Setup sudo

```
# nano /etc/sudoers

uncomment:
" %wheel ALL=(ALL) ALL"
```

Setup users and passwords

```
# useradd -m -G wheel {your_username}
# passwd {your_username}

# passwd (this will set the root password)
```

Install GRUB

```
# pacman -S grub efibootmgr os-prober
# grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck
# ls -l /boot/grub (confirm grub installed)
```

Configure GRUB

```
# nano /etc/default/grub

Then add: (Skip this part if you are not going to boot into Windows using the GRUB Menu)
# Enable os prober
GRUB_DISABLE_OS_PROBER=false

Finally If you are not booting into Windows using the GRUB Menu: (Optional)
Change "GRUB_TIMEOUT=5" to "GRUB_TIMEOUT=0" or "GRUB_TIMEOUT=1" depending on how long you wish to see the GRUB Menu at boot.
```

Make GRUB Config

```
# grub-mkconfig -o /boot/grub/grub.cfg
```

Exit and unmount

```
# exit
# umount -R /mnt
# reboot
```

Install Xorg server

```
# sudo pacman -S xorg-server
```

Install a graphics driver for your system

```
Check what graphics driver you need here:
https://wiki.archlinux.org/title/xorg#Driver_installation


Then install it:
sudo pacman -S {driver}

Note: If you have a NVIDIA GPU Install "nvidia-dkms" if you installed a kernel other than "linux" when running pacstrap.
```

Install GNOME (Optional)

```
# sudo pacman -S gnome gnome-tweaks xorg-server
# sudo systemctl enable gdm

Install a Browser and Terminal Emulator (Optional):
# sudo pacman -S firefox tilix

# reboot
```

Install Oh My Zsh (Optional)

```
# sudo pacman -S zsh
# sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Install yay (AUR Helper) (Optional)

```
# git clone https://aur.archlinux.org/yay.git
# cd yay
# makepkg -si
# yay -c (removes unneeded dependencies)
# cd ..
# rm -rf yay (removes yay directory)
```

Replace sudo with doas (Optional)

```
# sudo pacman -S opendoas

# sudo nano /etc/doas.conf
Then add:
permit persist :wheel

# doas pacman -R sudo
# doas ln -s /bin/doas /bin/sudo (symlink sudo to doas)
```
If you face any issues try consulting [this page](https://wiki.archlinux.org/title/Dual_boot_with_Windows)

And you're done! Hope this guide helped :)
