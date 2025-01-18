# Arch Install Notes

#### Links

[Change TTY Font](https://wiki.archlinux.org/title/Linux_console)

[Spotify-TUI](https://github.com/Rigellute/spotify-tui)

#### If you are installing on a real computer remember to install their microcode at install through intel and amd ucode officical packages

If you have a graphics card you must find a compatible driver for it and install to have it working.

#### Install a backup kernel

I recommend you just install your backup kernel first, make your grub config and then install the kernel you actually want to use after and after that make your grub config again as it will save you time attempting to make grub use the kernel you actually want by default instead of needing to go into `Advanced Options for Arch Linux` each boot.

Also this may not be worth the effort as you can just boot off of an arch iso and install a backup kernel or repair the old one when it is required.

```
Installing linux kernel:
# sudo pacman -S linux linux-headers
```

Note: Replace `zen` in the following command with `lts` or `hardened` if you do not wish to install the `zen` kernel.

```
Installing other officially supported kernels: (https://wiki.archlinux.org/title/kernel)
# sudo pacman -S linux-zen linux-zen-headers
```
