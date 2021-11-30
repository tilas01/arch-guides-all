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

## Setup DNS Caching with dnsmasq + Custom DNS Servers

#### Install dnsmasq

```
# sudo pacman -Sy dnsmasq
```

#### Configure NetworkManager to use dnsmasq

Create the following configuration file

`/etc/NetworkManager/conf.d/dns.con`

```
[main]
dns=dnsmasq
```

#### Enable IPv6 with dnsmasq

Create the following configuration file

`/etc/NetworkManager/dnsmasq.d/ipv6-listen.conf`

```
listen-address=::1
```

#### Enable DNSSEC with dnsmasq

Create the following configuration file

`/etc/NetworkManager/dnsmasq.d/dnssec.conf`

```
conf-file=/usr/share/dnsmasq/trust-anchors.conf
dnssec
```

#### Set Custom Global DNS Servers

Create the following configuration file

`/etc/NetworkManager/conf.d/dns-servers.conf`

```
[global-dns-domain-*]
servers={server1},{server2},{server3},{server4}
```

I personally use [Cloudflare DNS](https://1.1.1.1/dns/)

#### Restart NetworkManager

```
# sudo systemctl restart NetworkManager
```

#### Some Final Notes

If you are failing to connect try restarting your computer + the `NetworkManager` and `iwd` services. Then attempt to confirm your connection again.

[NetworkManager Arch Wiki Page](https://wiki.archlinux.org/title/NetworkManager)

[DNS Management with NetworkManager](https://wiki.archlinux.org/title/NetworkManager#DNS_management)

[iwd Arch Wiki Page](https://wiki.archlinux.org/title/Iwd#iwctl)
