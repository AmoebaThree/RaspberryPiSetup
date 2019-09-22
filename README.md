# Image

- Download Raspbian image, no need to unzip - [image download page](https://www.raspberrypi.org/downloads/)
- Download [Etcher](https://www.balena.io/etcher/)
- Plug in SD card
- Etch the zip file directly to the SD card
- Mount the card and create an empty file called "ssh" in the root (included in repo for Windows) - this enables SSH on first boot (see [here](https://www.raspberrypi.org/documentation/remote-access/ssh/))

[Source docs](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)

[Other options](https://www.raspberrypi.org/documentation/installation/installing-images/windows.md)

# First boot

- Plug into ethernet and SSH as user `pi` password `raspberry`
- Change the password - `passwd` and follow instructions
- Change the hostname to something sensible - `sudo raspi-config` in the networking menu
- Enable SSH permanently - `sudo raspi-config` and in the menu ([details](http://www.raspberrypi-spy.co.uk/2012/05/enable-secure-shell-ssh-on-your-raspberry-pi/))

Once networking setup, or if ethernet in
- `sudo apt-get update`
- `sudo apt-get install vim`
- `sudo apt-get autoremove`

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
- **Append** the contents of `networking/host/dhcpcd.conf` to `/etc/dhcpcd.conf`

- Reboot machine: `sudo shutdown -r now`

# Hardware

### Drive reset for initio

The Pirocon board has an annoying habit of driving the wheels on the first time it is powered on (restarting the Pi seems to be fine once the power is on).

- `mkdir ~/code/drivereset`
- Put the contents of the `drivereset` directory in there, except `drivereset.service`
- `cd ~/code/drivereset`
- Put `drivereset/drivereset.service` in `~/.config/systemd/user/`
- `sudo loginctl enable-linger pi`
- `systemctl --user enable drivereset.service`

[Guide](https://github.com/torfsen/python-systemd-tutorial)

### Servod for initio (servo endpoints only)

- `wget -q http://4tronix.co.uk/initio/servod.xxx -O servod`
- `chmod +x servod`

### PiFace

- Enable SPI in `sudo raspi-config`
- `pip install pifacedigitalio`
- `pip install pifacecommon`

### Infrared

https://help.ubuntu.com/community/LIRC
https://github.com/tompreston/python-lirc

### Bluetooth

https://ubuntuforums.org/showthread.php?t=1332197

### Raspberry Pi camera

https://picamera.readthedocs.io/en/release-1.13/recipes1.html

The sensor itself has a native resolution of 5 megapixel, and has a fixed focus lens onboard. In terms of still images, the camera is capable of 2592 x 1944 pixel static images, and also supports 1080p @ 30fps, 720p @ 60fps and 640x480p 60/90 video recording.

### Webcams

https://www.raspberrypi.org/documentation/usage/webcams/

# Software

### SSH

- Enable with `sudo raspi-config` in the networking section

### Share folders

- `sudo apt-get install samba samba-common-bin`
- `mkdir code`
- `chmod 777 code`
- `sudo smbpasswd -a pi` and add the password
- Put the contents of `smb.conf` at the bottom of `/etc/samba/smb.conf`
- Also in that file, comment out the bit about "homes" - this is spread over quite a few lines
- Restart with `sudo systemctl restart smbd.service`

[Example](http://raspberrypihq.com/how-to-share-a-folder-with-a-windows-computer-from-a-raspberry-pi/)

### Redis

On the host server:

- `sudo apt-get install redis-server`
- `sudo vim /etc/redis/redis.conf`
- In this file, set the "bind" directive to include `192.168.0.1`
- `sudo systemctl restart redis-server.service`

On any clients you want to try the CLI with:

- `sudo apt-get install redis-tools`

For any python clients:

- `pip install redis`

### Python

- `sudo apt-get install python-pip`

### Creating a userspace python systemd service

Template in the PythonService folder.

- `sudo apt-get install python-systemd`
- Copy the whole folder to /home/pi/code/PythonService
- Make a symlink to the service file: `ln -s ~/code/PythonService/service.service ~/.config/systemd/user/service.service`
- If you want to start it on boot, enable it with `systemctl --user enable service.service`
- Otherwise you can just start it with `systemctl --user start service.service`
- If you change the service file afterwards, you need to do `systemctl --user daemon-reload`

Based on [this guide](https://github.com/torfsen/python-systemd-tutorial)