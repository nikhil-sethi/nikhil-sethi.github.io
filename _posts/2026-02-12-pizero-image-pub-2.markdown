---
layout: post
title:  "Rpizero with ROS2 (?) image publisher (part 2)"
date:   2026-02-12
categories: networking raspberry-pi 
---


## Setting up usb internet (optional)

Wifi can be slow on the pi, so I wanted to setup internet via USB ethernet. This should usually work in the `Shared to other computers` configuration but it did not. I used [this](https://youtu.be/MJ084wtjiWM?t=335) video to setup ip table routes.

In summary, on your host device do:

```bash

echo 1 > /proc/sys/net/ipv4/forward

sudo iptables -t nat -I POSTROUTING -o 10.42.0.1/24 -j MASQUERADE

sudo iptables -I FORWARD -i enxce682bf756ab -j ACCEPT
sudo iptables -I FORWARD -o enxce682bf756ab -j ACCEPT


## verify (you should see the route with the correct interface)
sudo iptables -L  -n -v --line-numbers
```

Here `10.42.0.1/24` is the ip adress of host on the `enxce682bf756ab` USB ethernet interface. This will probably be different for you.

If everything went well, you should be able see pings when you do `ping 8.8.8.8`.

## Installing docker

Now you can run the instructions from [here](https://docs.docker.com/engine/install/raspberry-pi-os/#install-using-the-repository) and they should work successfully.
It worked for me and now I have docker up and running. Great.


## Running ros2 in docker