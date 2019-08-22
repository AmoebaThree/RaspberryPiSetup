# Image

- Download Raspbian image, no need to unzip - [image download page](https://www.raspberrypi.org/downloads/)
- Download Etcher - [site](https://etcher.io/)
- Plug in SD card
- Etch the zip file directly to the SD card
- Mount the card and create an empty file called "ssh" in the root (included in repo for Windows) - this enables SSH on first boot (see [here](https://www.raspberrypi.org/documentation/remote-access/ssh/))

[Source docs](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)

[Other options](https://www.raspberrypi.org/documentation/installation/installing-images/windows.md)

# First boot

- Plug into ethernet and SSH as user `pi` password `raspberry`
- Change the password - `passwd` and follow instructions
- Enable SSH permanently - `sudo raspi-config` and in the menu ([details](http://www.raspberrypi-spy.co.uk/2012/05/enable-secure-shell-ssh-on-your-raspberry-pi/))
- `sudo apt-get update`
- `sudo apt-get install vim`
- `sudo apt-get clean`

[Source docs](https://www.raspberrypi.org/documentation/linux/usage/users.md)

# Networking

### Host

- Put the contents of `networking/host/iptables.conf` in `/etc/iptables.conf`
- Put the contents of `networking/host/wlan0` in `/etc/network/interfaces.d/wlan0`
- Put the contents of `networking/host/wlan1` in `/etc/network/interfaces.d/wlan1`
- Put the contents of `networking/host/general` in `/etc/network/interfaces.d/general`
- **Append** the contents of `networking/host/dhcpcd.conf` to `/etc/dhcpcd.conf`
- `networking/host/wpa_supplicant-wlan1.conf` provides an example of how to specify wifi networks to connect to in `/etc/wpa_supplicant/wpa_supplicant-wlan1.conf`

- Install DHCP server: `sudo apt-get install isc-dhcp-server`
- Put contents of `networking/host/dhcpd.conf` in `/etc/dhcp/dhcpd.conf`
- Edit `/etc/default/isc-dhcp-server` and set INTERFACESv4="wlan0" and INTERFACESv6="wlan0"

- Install the AP server: `sudo apt-get install hostapd`
- Put the contents of `networking/host/hostapd.conf` in `/etc/hostapd/hostapd.conf`
- Change the # for the passphrase for the network
- Edit `/etc/default/hostapd` and point DAEMON_CONF to `/etc/hostapd/hostapd.conf`
- For some reason the service is masked in later raspbians:
`sudo systemctl unmask hostapd`
`sudo systemctl enable hostapd`
`sudo systemctl start hostapd`

- Reboot machine: `sudo shutdown -r now`

[Partial guide](https://learn.adafruit.com/setting-up-a-raspberry-pi-as-a-wifi-access-point/install-software)

[More](https://raspberrypihq.com/how-to-turn-a-raspberry-pi-into-a-wifi-router/)

[Angry, but helpful](https://tech.scargill.net/pi-zero-wi-fi-automatic-reconnect/)

[Probably the best docs](https://www.raspberrypi.org/documentation/configuration/wireless/access-point.md)

### Client

- Run `ifconfig` to check you have wlan0
- Put the contents of `networking/client/wlan0` in `/etc/network/interfaces.d/wlan0`
- Put the contents of `networking/client/general` in `/etc/network/interfaces.d/general`
- Replace the two #'s with the SSID and passphrase for your robot's network, and also its static IP number

- Reboot machine: `sudo shutdown -r now`

# Hardware

### Drive reset for initio

The Pirocon board has an annoying habit of driving the wheels on if connected at boot.

- `mkdir /usr/local/bin/drivereset`
- Put the contents of the `drivereset` directory in there (except `drivereset.service`)
- `cd /usr/local/bin/drivereset`
- `sudo wget -q http://4tronix.co.uk/initio/servod.xxx -O servod`
- `sudo chmod +x servod`
- Put `drivereset/drivereset.service` in `/etc/systemd/system`
- `sudo chmod 744 /usr/local/bin/drivereset/drivereset.sh`
- `systemctl daemon-reload`
- `systemctl enable drivereset.service`

# SSH

# Software

### Share folders

- `sudo apt-get install samba samba-common-bin`
- `mkdir code`
- `chmod 777 code`
- `sudo smbpasswd -a pi` and add the password
- Put the contents of `smb.conf` at the bottom of `/etc/samba/smb.conf`
- Also in that file, comment out the bit about "homes" - this is spread over quite a few lines
- Restart with `sudo /etc/init.d/samba restart`

[Example](http://raspberrypihq.com/how-to-share-a-folder-with-a-windows-computer-from-a-raspberry-pi/)

### Mono

### Redis

On the host server:

- `sudo apt-get install redis-server`
- `sudo vim /etc/redis/redis.conf`
- In this file, set the "bind" directive to include `192.168.0.1`
- `sudo service redis-server restart`

On any clients you want to try the CLI with:

- `sudo apt-get install redis-tools`