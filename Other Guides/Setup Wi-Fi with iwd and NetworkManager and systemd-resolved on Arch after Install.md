# Setup Wi-Fi with iwd and NetworkManager on Arch after Install

## In Arch chroot

#### Install NetworkManager and iwd

Do this while in arch-chroot or you will have to boot arch back up via the installation cd and install them

```
# pacman -S networkmanager iwd
```

#### Enable NetworkManager and iwd

```
# systemctl enable NetworkManager
# systemctl enable iwd
```

---

## In Arch Install

#### Set iwd as NetworkManager Wi-Fi backend

Create the following configuration file

`/etc/NetworkManager/conf.d/wifi_backend.conf`

```
[device]
wifi.backend=iwd
```

#### Restart NetworkManager and iwd

```
# sudo systemctl restart NetworkManager
# sudo systemctl restart iwd (may not be neccassary)
```

### Connect with nmcli ([ref](https://wiki.archlinux.org/title/NetworkManager#nmcli_examples))

```
# nmcli device wifi list
# nmcli device wifi connect "{ssid}" --ask
```

#### Confirm your Connection

```
# nmcli (confirm it says "Connected to {ssid} at the top")
# ping archlinux.org
```

---

## Setup Local DNS Caching, DoT, DoH + Custom DNS Servers and disable LLMR + Multicast with systemd-resolved

Note before we start you can also do this with dnsmasq and unbound also if you dont wish to use the systemd dns resolver. Check the arch wiki for intructions on how to setup these but otherwise this is absolutely fine if you're happy using systemd (which I am) not that I think systemd couldn't be better. But its reliable and the most used init system in the world, take from that what you will and review the differences you dont need to copy every decision i make but if you want to thats fine just letting you have the ability to use any dns resolver you want, its the beauty of arch. And admitly *most* linux distros also if you wish to its the beauty of unix and open source. And of course always feel free to follow your own steps and go off the guide with whatever you prefer if you do know what you are doing!

#### Install systemd-resolved (it should already be installed)

```
# pacman -Sy systemd-resolved
```

#### Configure NetworkManager to use systemd-resolved

Create the following configuration file

`/etc/NetworkManager/conf.d/dns.conf`

```
[main]
dns=systemd-resolved
```

#### Link system resolv.conf to systemd-resolved

`ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf`

#### Create config file for systemd-resolved

This file lets you configure all your DNS settings.
My config and all the settings I recommend you copy and enable will be listed and explained below.

Create the configuration file below

`/etc/systemd/resolved.conf`

This is the main config file for systemd-resolved. You can also choose to configure through seperate files but its this way or the other and this way is a lot easier and everything is in one place its all the same variables just spread out instead of all together in one text document which probably makes it more minimal also.

Its fine to use my format and your own configuration settings and any DNS provider other than Quad9 that supports the settings you enable such as DoT (DNS over TLS), although almost all DNS providers should meet your personal needs if you haven't picked one yet feel free to use mind as it is privacy focused and filters unwanted dns requests (visit quad9's website {add hyperlink for quad9 website link here} for more info).

List of DNS Settings my below config will apply:

Set main global DNS servers - "DNS={fallback servers}"

Set global fallback DNS servers - "DNS={fallback servers}"

Enable DNSSEC - "DNSSEC=true"

Enable DNS over TLS - "DNSOverTLS=yes" (required or need https)

Enable DNS Local Cacheing (for faster dns lookups) - "Cache=yes"

*The following will disable LLMR and Multicast DNS (keep in mind this is more of a security measure to stop other people on my network using me for dns and/or me using them it all stays in the setup you have created:*

"LLMR=no"

"MulticastDNS=no"

*Enable DNSStubListner - this will be a local proxy for all apps resulting in more compatibility*

```
[Resolve]
DNS={Quad9 - IPv4 DNS IP - Main}#{Dot Quad9 DNS IP} {Quad9 - IPv4 DNS IP - Fallback}#{Dot Quad9 DoT}
FallbackDNS={Quad9 - IPv4 DNS IP - Fallback} {Quad9 - IPv6 DNS IP - Fallback}
DNSSEC=true
DNSOverTLS=yes
DNSOverHTTPS=yes
Cache=yes
LLMR=no
MulticastDNS=no
DNSStubListner=yes
```

#### Enable and Start systemd-resolver

*Ensure you enable and Start systemd=resolved.service and NOT systemd-resolved.socket - this is why we are specying .service in the command below.. If left unspecified systemctl will assume you meant .service and execute as that and all should be fine. But because systemd-resolved has another one called systemd-resolved.socket im just specifying here to be safe. Realistically there should be no issue unless a command was run specifying to alter the systemd-resolved.socket command.*

```
# sudo systemctl enable systemd-resolved.service
# sudo systemctl start systemd-resolved.service
```

#### Restart NetworkManager

```
# sudo systemctl restart NetworkManager systemd-resolved
```

#### Check the DNSStubListner to ensure it is online, if its not reboot and look again.

Run the following command to confirm its listening on 127.0.0.53 and port 53

`ss -tulpn | grep 53`

This will confirm its hosted on 127.0.0.53:53

#### Some Final Notes

If you are failing to connect try restarting your computer + the `NetworkManager` and `iwd` services. Then attempt to confirm your connection again. I would say any issues here would be service related so just make sure all network services are enabled (networkmanager, iwd, systemd-resolved, etc) and then restart the system and your issues should be all fixed up. I hope this helped you!

[NetworkManager Arch Wiki Page](https://wiki.archlinux.org/title/NetworkManager)

[DNS Management with NetworkManager](https://wiki.archlinux.org/title/NetworkManager#DNS_management)

[iwd Arch Wiki Page](https://wiki.archlinux.org/title/Iwd#iwctl)
