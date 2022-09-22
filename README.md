# rtl8188eu-ur
Enable RTL8188EU compatible USB WIFI dongle for Universal Robot.

# DISCLAIMER
**USE AT YOUR OWN RISK. Be sure to make a backup of your robot first. THIS IS FOR RESEARCH PURPOSES ONLY.**

This document assumes you have
- root access on your UR robot
- working internet connection (on your robot)
- Compatible Wifi USB Dongle (e.g. [TP-Link TL-WN725N](https://www.amazon.de/-/en/TP-Link-TL-WN725N-Adaptor-Suitable-10-9-10-13) )

## Add debian repos
First, enable apt-get by adding the official repos for Depian Jessie
````
nano /etc/apt/sources.list
````

Add the appropriate mirrors for your location from https://linuxconfig.org/debian-apt-get-jessie-sources-list, eg
````
# /etc/apt/sources.list :
deb http://ftp.at.debian.org/debian/ jessie main contrib non-free
deb-src http://ftp.at.debian.org/debian/ jessie main contrib non-free
````

## Required packages
Next, install required packages
````
apt update
apt install git ifupdown iproute2 wpasupplicant iw wireless-tools
````

## Compile driver
Clone Branch 'v4.1.8_9499' from [lwfinger/rtl8188eu](https://github.com/lwfinger/rtl8188eu)
````
git clone -b v4.1.8_9499 https://github.com/lwfinger/rtl8188eu.git
````

Compile the driver
````
cd rtl8188eu/
make all
````

Install the driver
````
make install
````
If you get an error like
````
/lib/modules/[ur_kernel_version]/kernel/drivers/net/wireless: No such file or directory
````
just create the directory by manually before trying again
````
mkdir /lib/modules/[ur_kernel_version]/kernel/drivers/net/wireless
````

## Configure WiFi
from [Debian Wiki](https://wiki.debian.org/WiFi/HowToUse#Using_ifupdown)

Find your wireless interface and bring it up:
````
ip a
iw dev
ip link set wlan0 up
````
Now edit `/etc/network/interfaces`. The required configuration is much dependent on your particular setup. The following example will work for most commonly found WPA/WPA2 networks:
````
# my wifi device
allow-hotplug wlan0
iface wlan0 inet dhcp
        wpa-ssid ESSID
        wpa-psk PASSWORD
````

Bring up your interface and verify the connection:

````
ifup wlan0
iw wlan0 link
ip a
````

## Reboot
````
reboot now
````

# Show IP address after sucessfull WiFi connection
If you want to diplay the current IP adress on the teach pendant, create a new script in `/etc/network/if-up.d/` e.g. `show_wifi_ip`:
````
#! /bin/sh
# Get the current ip address of interface 'wlan0' and show it on the teach pendant

if [ "$IFACE" = "wlan0" ]; then
    ip4=$(/sbin/ip -o -4 addr list wlan0 | awk '{print $4}' | cut -d/ -f1)
    echo "WiFi IP: $ip4" | DISPLAY=:0 aosd_cat -R green -x 130 -y -210 -n "Arial Black 40"
fi
````
