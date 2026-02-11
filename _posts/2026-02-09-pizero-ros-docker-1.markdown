---
layout: post
title:  "Rpizero with ROS2 image publisher: Setting up docker (part 1)"
date:   2026-02-10
categories: networking raspberry-pi 
---

## Overview

The next goal is to hookup a camera to the rpi zero and stream images via the camera on the wifi/usb network via micro-ros.
I have the following steps in mind

1. Setup docker: We do things a bit more advanced here now and docker would really help in managing the nodes on the pi. It would also help in saving images and reusing later. Dependency managemetn is annoying on embedded devices. But i already know it's going to be a pain. LETS GO.

2. Make/Get the ros-perception container. This should have opencv, image publishers etc.. This is a bit shaky because I don't even know if full ROS will run on the rpi zero. Maybe we need micro-ros. Who knows. LETS GO

3. Publish images on the network and test the rates on the host via wifi. This requires me to be on the same network and listening on a ros2 node on my laptop as well. Totally doable. No problem. very easy. just having fun. LETS GOOOOO.


## Docker on the raspberry pi zero.

I ran the guide [here](https://docs.docker.com/engine/install/raspberry-pi-os/#os-requirements) and it could not find the packages. According to that page, raspi os (trixie) is not supported anymore and I'll probably need bookworm.

I also tried the script at https://get.docker.com but got the same error. No surprises there.

I also already tried installing the debian binaries for trixie from [here](https://docs.docker.com/engine/install/debian/) and it installed docker succesfully but it did not work at runtime. Gave a `Segmentation fault`. Classic. This is probably because I'm trying to run Debian on raspbian.

### Update 11/02: Going back to OS/Network config

So i tried installing bookworm (many times) and and it turns out that USB gadget mode does not work properly in it because the `usb0` does not come up by default. See [this topic](https://forums.raspberrypi.com/viewtopic.php?t=357938). As a result the host side does not create the Ethernet connection as well.

To solve this, we need to let nmcli take care of connection management which is doesn't work by default in RaspiOS Bookworm.

When you install the OS on the SD card, add the following lines to the `/etc/NetworkManager/NetworkManager.conf` file.
> If this file does not exist yet, first boot up the pi normally, connect via WiFi using ssh and then change the file.

```ini
[device]
match-device=interface-name:usb0
managed=1
```

You can now reboot to let nmcli manage (and bringup) the USB ethernet connection for you. Note that at this stage this connection will be named 'Wired Connection 1' but we can change this later in the following steps. In case this name is different for you, change it in the command. 

### Add a static IP (Optional)

```bash
# Add a static Ip to the connection
sudo nmcli connection modify Wired\ connection\ 1 connection.id usb0 ipv4.addresses 10.42.0.10/24 ipv4.gateway 10.42.0.1 ipv4.dns 10.42.0.1 ipv4.method manual

# Restart to make settings persistent
sudo nmcli connection down usb0
sudo nmcli connection up usb0
```

**Verify (Optional)**

These connection details should be stored in conf files. You can see these files via the following command:
`sudo  nmcli -f NAME,DEVICE,FILENAME connection show`

```bash
NAME           DEVICE  FILENAME                                                          
usb0           usb0    /etc/NetworkManager/system-connections/usb0.nmconnection          
preconfigured  wlan0   /etc/NetworkManager/system-connections/preconfigured.nmconnection 
lo             lo      /run/NetworkManager/system-connections/lo.nmconnection 
```

`cat /etc/NetworkManager/system-connections/usb0.nmconnection` should show the details of the ip address you set.

Now everytime you reboot the pi, you should see the Ethernet connection on the host with static ip on the pi.

FINALLY. I'm gonna try setting up docker AGAIN tomorrow. See you then!