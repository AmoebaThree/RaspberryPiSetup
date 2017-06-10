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
- `sudo apt-get remove wolfram-engine`
- `sudo apt-get install vim`
- `sudo apt-get upgrade`
- `sudo apt-get dist-upgrade`
- `sudo apt-get autoclean`
- `sudo apt-get clean`

[Source docs](https://www.raspberrypi.org/documentation/linux/usage/users.md)

# Networking

### Host

- Run `ifconfig` to check you have wlan0 and wlan1
- Turn off wlan0: `sudo ifdown wlan0`
- Turn off wlan1: `sudo ifdown wlan1`

- Edit `/etc/sysctl.conf` and add `net.ipv4.ip_forward=1`
- Put the contents of `iptables.ipv4.nat` in `/etc/iptables/ipv4.nat`

- Put the contents of `host.interfaces` in `/etc/network/interfaces`
- Replace the two #'s with the SSID and passphrase for your passthrough network

- Install the AP server: `sudo apt-get install hostapd`
- Put the contents of hostapd.conf in `/etc/hostapd/hostapd.conf`
- Change the # for the passphrase for the network
- Edit `/etc/default/hostapd` and point DAEMON_CONF to `/etc/hostapd/hostapd.conf`

- Reboot machine

[Partial guide](https://learn.adafruit.com/setting-up-a-raspberry-pi-as-a-wifi-access-point/install-software)

### Client

- Run `ifconfig` to check you have wlan0
- Turn off wlan0: `sudo ifdown wlan0`

- Put the contents of `client.interfaces` in `/etc/network/interfaces`
- Replace the two #'s with the SSID and passphrase for your robot's network, and also its static IP number

- Reboot machine

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