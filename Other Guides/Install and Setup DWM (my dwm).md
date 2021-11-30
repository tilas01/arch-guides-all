# Install and Setup DWM on Arch (my dwm)

Install yay (AUR Helper)

```
# git clone https://aur.archlinux.org/yay.git
# cd yay
# makepkg -si
# yay -c (removes unneeded dependencies)
# cd ..
# rm -rf yay (removes yay directory)
```

#### Replace sudo with doas

```
# sudo pacman -Sy opendoas

# sudo nano /etc/doas.conf
Add:
permit persist :wheel

# sudo pacman -R sudo
# doas ln -s /bin/doas /bin/sudo (symlink sudo to doas)
```

#### Install Dependencies and Some Other Applications

```
# sudo pacman -Sy xorg-server xorg-xrandr xorg-xinit xorg-xset libx11 libxft libxinerama xautolock ttf-jetbrains-mono git python-pywal feh ttf-font-awesome ranger alsa-utils pulseaudio firefox btop
```

#### Create a Directory for DWM

```
# mkdir ~/git (you can make this any filename you like but for this guide i will be using "git")
# cd ~/git
```

#### Clone DWM and Wallpapers (Desktop)

```
# git clone https://github.com/tilas01/dwm.git
# git clone https://github.com/tilas01/st.git
# git clone https://github.com/tilas01/dmenu.git
# git clone https://github.com/tilas01/slstatus.git
# git clone https://github.com/tilas01/slock.git
# git clone https://github.com/tilas01/wallpapers.git
# git clone https://github.com/tilas01/dotfiles.git (intended for laptop)
```

#### Clone DWM and Wallpapers (Laptop)

```
# git clone -b laptop https://github.com/tilas01/dwm.git
# git clone https://github.com/tilas01/st.git
# git clone https://github.com/tilas01/dmenu.git
# git clone -b laptop https://github.com/tilas01/slstatus.git
# git clone https://github.com/tilas01/slock.git
# git clone https://github.com/tilas01/wallpapers.git
# git clone https://github.com/tilas01/dotfiles.git
```

#### Make DWM

```
# cd dwm
# sudo make clean install
# cd ../st
# sudo make clean install
# cd ../dmenu
# sudo make clean install
# cd ../slstatus
# sudo make clean install
# cd ../slock
# sudo make clean install
```

#### Add DWM to .xinitrc

```
# sudo cp /etc/X11/xinit/xinitrc ~/.xinitrc
# sudo nano ~/.xinitrc
Remove "twm &" and all lines below it.
Then add:
slstatus &
xautolock -time 10 -locker slock -corners 000- -detectsleep &
exec dwm

Note: Don't add the xautolock line if you don't want the system to automatically lock after 10 minutes unless mouse is in the bottom right corner. And remove/don't install the package if it is not being used.
```

#### Add xrandr to .xinitrc (Optional) - (Only if you're on a VM)

```
# sudo nano ~/.xinitrc
Add to it before "slstatus &":
xrandr --output Virtual-1 --mode 1920x1080

Note: Replace "Virtual-1" with your display name if it isn't that
```

#### Add startx to your .bashrc

```
# sudo nano ~/.bashrc
Add to the start of it:
if [ -z "${DISPLAY}" ] && [ "${XDG_VTNR}" -eq 1 ]; then
  exec startx
fi
```

#### Reboot

```
# reboot
```

#### Set slock to run on sleep

Create the following configuration file

`/etc/systemd/system/slock@.service`

```
[Unit]
Description=Lock X session using slock for user %i
Before=sleep.target

[Service]
User=%i
Environment=DISPLAY=:0
ExecStartPre=/usr/bin/xset dpms force suspend
ExecStart=/usr/local/bin/slock

[Install]
WantedBy=sleep.target
```

```
# sudo systemctl enable slock@{username}.service
# systemctl suspend (this will sleep your system - confirm slock runs)
```

#### Block access to TTY and Prevent a user from killing X to prevent bypassing slock

Create the following configuration file

`/etc/X11/xorg.conf.d/slock.conf`

```
Section "ServerFlags"
    Option "DontVTSwitch" "True"
    Option "DontZap"      "True"
EndSection
```

```
[Alt]+[Shift]+[q] (quit dwm)
Now log back in and run the following commands to confirm the changes worked.

# sudo cat ~/.local/share/xorg/Xorg.0.log | grep "VT\|ZAP"

If you're output looks something like this you're good:
[  1144.672] (**) Option "DontVTSwitch" "True"
[  1144.672] (**) Option "DontZap" "True"
[  1144.673] (++) using VT number 1
```

#### Set Wallpaper

```
# feh --bg-scale ~/git/wallpapers/cat_town.png
```

#### Add feh to .xinitrc

```
# sudo nano ~/.xinitrc
Then add before "exec dwm":
~/.fehbg &
```

#### Enable pywal and add it to .bashrc

```
# wal -i ~/git/wallpapers/cat_town.png -n

Add to the bottom of .bashrc:
cat ~/.cache/wal/sequences
```

#### dwm note

```
Change font size to 16-18 (16 seems best) if installing on laptop (both font variables at beginning)
```

#### Create Directories (Optional)

```
# cd ~
# mkdir Documents Pictures Videos Downloads
```

#### Install pfetch

```
Manually:
# cd ~/git
# git clone https://github.com/dylanaraps/pfetch
# cd pfetch
# chmod +x pfetch
# sudo cp pfetch /usr/bin
# cd ..
# rm -rf pfetch

With yay:
# yay -S pfetch
or
# yay -S pfetch-git
```

Install cava

```
# yay -S cava
```

#### Install Oh My Zsh (Optional)

```
# sudo pacman -S zsh
# sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

1. Add volume control to DWM with Alsa (alsa-utils)
2. Add brightness control to DWM with xrandr or another tool
3. Customize dmenu a bit more
4. Make a keyboard shortcut for firefox

#### Setup btop

Launch `btop` and press `esc` then select `Options` and set `Theme Background` to `False`

#### How to change theme

```
# cd ~/git/wallpapers
# pywal -i (wallpaper)
# feh --bg-scale (wallpaper)

edit dwm config.h and change col_cyan[] to a matching color with the wallpaper
```

#### Patching Suckless Programs

```
Recommended (best) way:
# patch < the-patch.diff

If that way fails: (idk what the difference is)
# patch -p1 < the-patch.diff

Last resort
# git apply the-patch.diff

If none of that works go to the .rej file and manually apply the patch where it failed.
```

#### How to git pull from laptop after pushing changes from Arch VM
```
# git reset --hard origin/main
# git pull

If on dwm change both the font sizes back from "12" to "16" (or 18 but 16 is preferred)

Note: these commands may have to be the other way around I guess we'll see.
```

#### Some Final Notes

I personally think Mental Outlaw has some great examples for configuring these programs at his [GitHub](https://github.com/MentalOutlaw?tab=repositories) and on his [YouTube](https://www.youtube.com/channel/UC7YOGHUfC1Tb6E4pudI9STA) so check that out if you want.

[DWM Keybind Cheat Sheet](https://ratfactor.com/dwm)

[Official DWM Tutorial](https://dwm.suckless.org/tutorial/)

[Another Keybind Cheat Sheet](https://gist.github.com/erlendaakre/12eb90eef84a3ab81f7b531e516c9594)

[My Dotfiles (for laptop - if using a vm add the xrandr line to .xinitrc)](https://github.com/tilas0/dotfiles)

[How to Patch Suckless Programs](https://suckless.org/hacking/)

[Video on How to Rice DWM by Mental Outlaw](https://youtu.be/DlViR5Ymg4A)

[Video on How to Patch Suckless Programs by Mental Outlaw](https://youtu.be/80FISSjYYrk)

[Video on How to Patch Suckless Programs by DistroTube](https://youtu.be/3dwoC0EYStw)

[Patch DWM in 3 different ways](https://youtu.be/BFIR3tQQ4cQ)

[Video on SLStatus by Mental Outlaw](https://youtu.be/nl6TeGPJBV0)

[My Thoughts on Suckless](https://youtu.be/wgRZNPxErEs)

[Setup Vim-Plug](https://www.chrisatmachine.com/Neovim/01-vim-plug/)

[How to Secure your System](https://wiki.archlinux.org/title/security)

[Security for USB Ports](https://wiki.archlinux.org/title/USBGuard#Installation)

[feh Arch Wiki Page](https://wiki.archlinux.org/title/feh)

[Where I Got My Wallpapers](https://github.com/marsupial-king/my-arch-dots)

```
Commands:

Sleep: systemctl suspend (Create a Keybind for this)
Find Display Name: xrandr
Adjust Brightness: xrandr --output {display} --brightness {brightness} (Create a Keybind for this)
Volume Mixer: alsamixer (to use fn keys have to press [fn]+[key])
Adjust Volume: (to-do) (Create a Keybind for this)

also add volume to slstatus?

default brightness is 1.0 but i set mine currently to 0.8
```

