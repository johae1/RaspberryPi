# Raspberry Pi 5 - Power and Networking over USB C

This document is for personal use only. It documents, how to configure a raspberry pi to communicate with an ipad pro.

## Raspberry Pi OS Lite (64-bit)

### Image Creation

1. download and install [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. choose model `Raspberry Pi 5`
3. select `Raspberry Pi OS Lite (64-bit)`
4. insert SD-card and select it
5. preconfigure these OS settings:
    - hostname (e.g. 'rpi5')
    - username and password
    - wifi settings (optional, if you use ethernet)
    - language and keyboard layout (optional)
    - **activate ssh (password auth)**
7. start imaging process
8. remove SD-card when finished

### Configure SSH

1. insert SD-card into Raspberry Pi 5
2. copy your public key to raspberry pi with the following command: 
```
ssh-copy-id <key-name> <username>@<hostname>.local
```
3. connect over ssh with the following command:
```
ssh <username>@<hostname>.local
```
4. edit the file `/etc/ssh/sshd_config` and add the following
```
PermitRootLogin no
PasswordAuthentication no
```
5. restart sshd-service with the following command:
```
sudo systemctl restart sshd
```

### Enable Networking over USB C

1. Edit the file `/boot/firmware/cmdline.txt` and add the following just AFTER `rootwait`:
```
modules-load=dwc2,g_ether
```
2. Edit the file `/boot/firmware/config.txt` and confirm it has an UNCOMMENTED line near the end that is `otg_mode=1`. Then add below the line `[all]` (it HAS to be after the `[all]` line for it to work properly) the following:
```
dtoverlay=dwc2
```
3. Now we need to add our connection name by running the following command:
```
sudo nmcli con add type ethernet con-name ethernet-usb0
```
4. Now edit the file we just created named `/etc/NetworkManager/system-connections/ethernet-usb0.nmconnection`
    - You will be adding the lines for `autoconnect` and `interface-name`, and then modifying the line with `method=` to change `auto` to `shared`:
```
[connection]
id=ethernet-usb0
uuid=<random group of characters here>
type=ethernet
autoconnect=true
interface-name=usb0
  
[ethernet]
  
[ipv4]
method=shared
  
[ipv6]
addr-gen-mode=default
method=auto
  
[proxy]
```
5. Create yet another new file at `/usr/local/sbin/usb-gadget.sh` with the following contents:
```sh
#!/bin/bash

nmcli con up ethernet-usb0
```
6. Make this new file executable with the following command:
```
sudo chmod a+rx /usr/local/sbin/usb-gadget.sh
```
7. Create your last new file named `/lib/systemd/system/usbgadget.service` with the following text:
```
[Unit]
Description=My USB gadget
After=NetworkManager.service
Wants=NetworkManager.service
  
[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/local/sbin/usb-gadget.sh
  
[Install]
WantedBy=sysinit.target
```
8. Run the following command to enable this new service:
```
sudo systemctl enable usbgadget.service
```
9. Reboot your pi5 so all your changes can be put to use. Run the following command to reboot it:
```
sudo shutdown -r now
```
