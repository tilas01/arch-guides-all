#### Created by [tilas01](www.github.com/tilas01) on GitHub
Do not remove my credit if sharing, modifying or publishing anything with this guide please.

With the above aside lets get started!
I hope this or any of my other guides can help you!

This guide and all information included in this repo is based on the official [
Arch Wiki](https://wiki.archlinux.org/title/Main_page)

# Setup Secure Boot with Shim (GRUB)

#### Assumptions

```
This guide assumes:
You are on a pre-existing Arch Linux Install
You have yay installed
You are using GRUB as your bootloader
Your efi partition is /efi
Your efi partition is your first partition
Your disk is /dev/sda

Replace these/alternate steps where applicable in this guide
```

#### Some Important Information

```
THIS GUIDE WILL NOT PROTECT YOU AGAINST AN EVIL MAID ATTACK

This guide is really only a work around for Secure Boot so I could dual-boot Arch and Windows 11 (Bitlocker) without having to disable Secure Boot everytime I wanted to switch to Arch.
Anyone could still just enroll their own hash in MokManager and boot even if you followed all the steps in this guide.

Review this wikipedia page before proceeding to understand what an Evil Maid attack is incase you are unaware or do not have a full understanding:
https://en.wikipedia.org/wiki/Evil_maid_attack

Real protection against an Evil Maid attack comes with using encryption and setting a Boot and Administrator (Setup Utility) password within your Setup Utility.
Essentially meaning if someone wanted to compromise your EFI Partition it would require physical extraction of your storage device.
So even if ALL the above steps are met it is simply just difficult, not impossible, to perform an Evil Maid attack.

If you REALLY want to protect your system from an Evil Maid attack with Secure Boot this IS NOT the guide for you.
I recommend you check out https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot.
It doesn't cover everything but can be a great starting point if this is what you want.

I could use my own keys but I don't want to deal with the risk of potentially bricking my hardware or facing other complications.
And Personally I doubt I would be a target for any form of sophisticated Evil Maid attack.
Anyway before proceeding with using your own keys I recommend you think about if the risks outweight the reward.

At the end of the day you are still susceptible to an Evil Maid attack even if you use your own keys.
All methods above and commonly implemented are really only considered preventative as it is extremely hard to stop an attacker with physical access.

Please correct me if any information in this guide is incorrect, outdated or could be improved.
```

#### Create SBAT

`sbat.csv`

```
sbat,1,SBAT Version,sbat,1,https://github.com/rhboot/shim/blob/main/SBAT.md
grub,1,Free Software Foundation,grub,2.04,https://www.gnu.org/software/grub/
```

#### Reinstall GRUB with SBAT and TPM

```
# sudo grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck --sbat=sbat.csv --modules="tpm"
# rm sbat.csv (optional)
```

#### Install Shim

```
# yay -S shim-signed
# sudo cp /usr/share/shim-signed/shimx64.efi /efi/EFI/GRUB/shimx64.EFI
# sudo cp /usr/share/shim-signed/mmx64.efi /efi/EFI/GRUB/
```

#### Create Keys

```
# sudo pacman -S sbsigntools
# sudo mkdir /root/sbkeys
# sudo su
# cd /root/sbkeys
Note: Replace "{common_name}" in the following command with the common name you want to use.
# openssl req -newkey rsa:4096 -nodes -keyout MOK.key -new -x509 -sha256 -days 3650 -subj "/CN={common_name}/" -out MOK.crt
# openssl x509 -outform DER -in MOK.crt -out MOK.cer
```

#### Copy MOK.cer to the EFI Partition

This is to Enroll with MokManager later.

```
# cp MOK.cer /efi
```

#### Sign GRUB and Kernel

```
# sbsign --key MOK.key --cert MOK.crt --output /boot/vmlinuz-* /boot/vmlinuz-*
# sbsign --key MOK.key --cert MOK.crt --output /efi/EFI/GRUB/grubx64.efi /efi/EFI/GRUB/grubx64.efi
```

#### Create pacman hooks to sign GRUB and Kernel

```
# mkdir /etc/pacman.d/hooks
```

`/etc/pacman.d/hooks/999-sign_kernel_for_secureboot.hook`

```
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = linux
Target = linux-lts
Target = linux-hardened
Target = linux-zen

[Action]
Description = Signing Kernel for Secure Boot
When = PostTransaction
Exec = /usr/bin/find /boot/ -maxdepth 1 -name 'vmlinuz-*' -exec /usr/bin/sh -c 'if ! /usr/bin/sbverify --list {} 2>/dev/null | /usr/bin/grep -q "signature certificates"; then /usr/bin/sbsign --key /root/sbkeys/MOK.key --cert /root/sbkeys/MOK.crt --output {} {}; fi' \ ;
Depends = sbsigntools
Depends = findutils
Depends = grep
```

`/etc/pacman.d/hooks/998-sign_grub_for_secureboot.hook`

```
[Trigger]
Operation = Install
Operation = Upgrade
Type = Package
Target = grub

[Action]
Description = Signing GRUB for Secure Boot
When = PostTransaction
Exec = /usr/bin/find /efi/ -name 'grubx64*' -exec /usr/bin/sh -c 'if ! /usr/bin/sbverify --list {} 2>/dev/null | /usr/bin/grep -q "signature certificates"; then /usr/bin/sbsign --key /root/sbkeys/MOK.key --cert /root/sbkeys/MOK.crt --output {} {}; fi' \ ;
Depends = sbsigntools
Depends = findutils
Depends = grep
```

#### Add Shim to EFI Boot Manager

```
# efibootmgr --verbose --disk /dev/sda --part 1 --create --label "Shim" --loader /EFI/GRUB/shimx64.EFI
```

#### Enable Secure Boot

```
# exit
# shutdown now
```

1. Boot into your `BIOS Setup Utility` and Enable `Secure Boot`.
2. Set an `Administrator Password` for your `BIOS Setup Utility` so not anyone can just disable `Secure Boot`.
3. Consider enabling a `Boot Password` if you have the option.
4. Save changes and reboot.

#### Enroll Key in MokManager

In MokManager select `Enroll key from disk` then find `MOK.cer` and Enroll it. Once your key is enrolled select `Reboot`.

#### Skip GRUB Boot Menu (Optional)

I dual boot Arch and Windows 11 with BitLocker which requires me to use secure boot. So for my use case I use my BIOS boot manager to select which operating system I want to use as shim does not allow chain loading. Meaning I have almost no use for the GRUB boot menu. So this is how to disable it or set it to 1 second to give you a chance if you need to do anything.

```
# sudo nano /etc/default/grub
Then change "GRUB_TIMEOUT=5" to "GRUB_TIMEOUT=0" or "GRUB_TIMEOUT=1" depending on how long you wish to see the GRUB menu.

Also you may want to replace "GRUB_TIMEOUT_STYLE=menu" with "GRUB_TIMEOUT_STYLE=hidden" if you set "GRUB_TIMEOUT=1".
This means you will have 1 second to press a key before grub boots but no menu will appear.

Also if you have a "GRUB_DISABLE_OS_PROBER=false" line, I recommend you remove it as this will skip that menu anyway

# sudo grub-mkconfig -o /boot/grub/grub.cfg
# reboot (confirm it worked)
```

Now to select whether I want to boot into Arch or Windows I simply have to press `F9` after entering my boot password and use the BIOS Boot Menu.

Note: Your Boot Manager key may be different from mine or your motherboard could lack that functionality.

#### Final Notes

If you face any issues I recommend you check out the [Arch Wiki](https://wiki.archlinux.org/), specifically [this page](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot).

And you're done. I hope this guide helped :)

