#### Created by [tilas01](https://www.github.com/tilas01) on GitHub
Do not remove my credit if sharing, modifying or publishing anything with this guide please.

With the above aside lets get started!
I hope this or any of my other guides can help you!

This guide and all information included in this repo is based on the official [
Arch Wiki](https://wiki.archlinux.org/title/Main_page)

# Install and Setup DWM on Arch

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

#### Install Dependencies and Git

```
# sudo pacman -Sy xorg-server xorg-xrandr xorg-xinit xorg-xset libx11 libxft libxinerama ttf-jetbrains-mono git btop
```

#### Create a Directory for DWM

```
# mkdir ~/git (you can make this any filename you like but for this guide i will be using "git")
# cd ~/git
```

#### Clone DWM

```
# git clone git://git.suckless.org/dwm
# git clone git://git.suckless.org/st
# git clone git://git.suckless.org/dmenu
# git clone git://git.suckless.org/slstatus
# git clone git://git.suckless.org/slock
```
#### Change slock config.h values to your username
```
# sudo nano config.h
Replace all instances of "nobody" and "nogroup" with your username to make slock work before building it.
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

#### Add DWM and SLStatus to .xinitrc

```
# sudo cp /etc/X11/xinit/xinitrc ~/.xinitrc
# sudo nano ~/.xinitrc
Remove "twm &" and all lines below it.
Then add:
slstatus &
exec dwm
```

#### Add xrandr to .xinitrc (Optional) - (Only if you're on a VM)

```
# sudo nano ~/.xinitrc
Add to it before "slstatus &":
xrandr --output Virtual-1 --mode 1920x1080

Note: Replace "Virtual-1" with your display name if it isn't that
```

#### Add startx to your .bash_profile

```
# sudo nano ~/.bash_profile
Add to the end of it:
if [ -z "${DISPLAY}" ] && [ "${XDG_VTNR}" -eq 1 ]; then
  exec startx
fi
```
#### Or if you are using OhMyZsh:
#### Add startx to .zshrc
```
sudo nano ~./zshrc
Add to the end of it:
if [ -z "${DISPLAY}" ] && [ "${XDG_VTNR}" -eq 1 ]; then
  exec startx
fi
```

#### Reboot

```
# reboot
```

#### Config

```
Everything from here is simply personal preference for me all I currently do is:
Set my ST Font to "JetBrains Mono" and Font(Pixel) Size to "20"
Set my DWM "*fonts[]" size to "12" or "15" if I'm on my laptop
Set my DWM "dmenufont[]" size to "12" or "14" if I'm on my laptop
Set "static const float mfact" under "layout(s)" to "0.50" in DWM
Set "static const char col_cyan[]" to "#de4e4e" (red) in DWM

removed default and add these lines to slstatus:
{ cpu_perc, "[CPU: %s%%]   ", NULL	      },
{ ram_perc, "[RAM: %s%%]   ", NULL	      },
{ datetime, "%s",           "%a %b %d %r" },

I do intend to customise it more, apply patches and make my own git repo for dwm later on.
```

#### Setup slock

Make the following changes to your slock config:

```
Set "*user" to your username
Set "*group" to your username
```

```
# sudo make clean install
# slock (confirm it worked)
```

#### Make [Alt]+[Shift]+[l] run slock

Make the following changes to your DWM config:

```
Add to the bottom of commands:
static const char *lockcmd[] = { "slock", NULL };

Add to keys[] under the termcmd entry:
{ MODKEY|ShiftMask,	XK_l,	spawn,	{.v = lockcmd } },
```

```
# sudo make clean install
```

```
[Alt]+[Shift]+[q] (quit dwm)
Then log back in and confirm that when you press [Alt]+[Shift]+[l] slock launches
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

#### Enable automatic locking after 10 minutes with xautolock (Optional)

```
# sudo pacman -S xautolock
# sudo nano ~/.xinitrc

Add this after "slstatus &":
xautolock -time 10 -locker slock -corners 000- -detectsleep &

This will automatically lock your system after 10 minutes of inactivity but will not lock if your cursor is in the bottom right corner.
You can remove or edit this option but I recommend keeping it as it can be convenient when watching a movie or video.
It is also possible with a bit of tweaking to make xautolock not lock when listening to audio but that won't be covered here.
Or alternatively you can use xidlehook(https://github.com/jD91mZM2/xidlehook).
```

```
[Alt]+[Shift]+[q] (quit dwm)
Now log back in and xautolock should be working in the way described above if you didn't modify the command.
```

```
# reboot
```

#### Create Directories (Optional)

```
# cd ~
# mkdir Documents Pictures Videos Downloads
```

#### Install Some Other Applications

```
# sudo pacman -Sy ranger feh alsa-utils pulseaudio
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

#### Install Oh My Zsh (Optional)

```
# sudo pacman -S zsh
# sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

#### Install Firefox

```
# sudo pacman -S firefox
```

#### Uninstall DWM + Revert steps in this guide (Obviously don't do this now)

```
# cd ~/git
# cd dwm
# sudo make clean uninstall
# cd ../st
# sudo make clean uninstall
# cd ../dmenu
# sudo make clean uninstall
# cd ../slstatus
# sudo make clean uninstall
# cd ../slock
# sudo make clean uninstall

# cd ..
# rm -rf dwm st dmenu slstatus slock

# sudo rm ~/.xinitrc
OR
Remove all the lines in ~/.xinitrc that you added while following this guide/that are related to dwm.

# nano ~/.bashrc
Remove:
if [ -z "${DISPLAY}" ] && [ "${XDG_VTNR}" -eq 1 ]; then
  exec startx
fi

# sudo systemctl disable slock@{username}.service
# sudo rm /etc/systemd/system/slock@.service
# sudo rm /etc/X11/xorg.conf.d/slock.conf
# reboot

I recommend you go through and manually remove any packages you do not require that were installed while following this guide.
And remove any files or folders you created relating to dwm.
```

#### To-Do

1. Make Repos for DWM
2. Add RAM and CPU and Volume to slstatus
3. Add volume control to DWM with Alsa (alsa-utils)
4. Customize dmenu a bit more
5. Make a keyboard shortcut for firefox

#### Some Final Notes

I personally think Mental Outlaw has some great examples for configuring these programs at his [GitHub](https://github.com/MentalOutlaw?tab=repositories) and on his [YouTube](https://www.youtube.com/channel/UC7YOGHUfC1Tb6E4pudI9STA) so check that out if you want.

[DWM Keybind Cheat Sheet](https://ratfactor.com/dwm)

[Official DWM Tutorial](https://dwm.suckless.org/tutorial/)

[Another Keybind Cheat Sheet](https://gist.github.com/erlendaakre/12eb90eef84a3ab81f7b531e516c9594)

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

```
Commands:

Sleep: systemctl suspend (Create a Keybind for this)
Find Display Name: xrandr
Adjust Brightness: xrandr --output {display} --brightness {brightness} (Create a Keybind for this)
Adjust Volume: (to-do) (Create a Keybind for this)
```

