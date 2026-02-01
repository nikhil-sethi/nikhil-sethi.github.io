---
layout: post
title: "Decentralized drone swarms: Part 1 (hardware setup)"
date: 2021-03-08
---

<html>

<body class="stackedit">
    <div class="stackedit__html">
        <p>Setting up raspberry pi’s networking was a pain.</p>
        <p>I know. It couldn’t get any easier and the developers should be commended on their work. And I am commending
            them. The raspberry pi is marvelous little computer capable of so much and it’s super easy to setup.</p>
        <p>That is- when you’re not completely oblivious about the very basics of networking and electronics and have a
            mind bending OCD to get everything perfect in the most optimized manner. Or in other words you’re unlike me.
        </p>
        <p>It was good pain though. I learnt a lot through it and did get it working at the end. So let’s start.</p>
        <p>My aim was to setup two headless raspberry pi zero W’s (v1.1) to communicate with each other over local
            decentralized ad-hoc network. My first thought was to check feasibility for the hardware. To be on the safe
            side I bought two ESP-01(esp8266) modules just in case the inbuilt wifi doesn’t work.</p>
        <p>That is most definitely a big task and I’ll cover it in further posts. For this post however we’ll be doing
            the following</p>
        <ol>
            <li>List and get hardware</li>
            <li>Connect two raspberry pi’s to the host pc via both wifi and usb over ethernet for maintenance purposes.
            </li>
        </ol>
        <p><a href="https://www.blogger.com/#">
                <div style="text-align: center;"><img alt=""
                        src="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/raspis.png"
                        title="No. Lenovo doesn't sponsor me." /></div>
            </a></p>
        <p>A brief look at my hardware:</p>
        <p>1 x Host PC: Lenovo Legion 5 AMD with Ubuntu 20.04 (5.8.4 generic )<br />
            2 x Raspberry pi zero W (v 1.1) with RaspiOS buster 11-01-2021)<br />
            1 x Raspberry pi 3 b<br />
            2 x <strong>good quality</strong> micro USB 2.0 cables with data transfer</p>
        <p>If you’re on a windows machine, things will work but will take some extra effort. There is existing
            documentation for it. If you can and know what you’re doing, shift to a linux-windows dual setup.</p>
        <p><a href="https://www.blogger.com/#">This</a> blog post with BATMAN-ADV and openWRT caught my attention. It
            said that not all firmware and protocols are compatible with mesh networking and the 802.11 s/r protocol
            might be needed to set it up. The wifi adapter in the rpi zero was b/n/g compatible which made me doubt the
            process but I still went ahead anyway. The problem occurred when I couldnt get the openWRT OS running on the
            rpi. For some reason the rpi would not complete the boot sequence. Apparently the reason is that the openwrt
            firmware waits for an enter key sequence the first time it boots without taking into account the
            headless/headed nature of the pi. Mine was headless so clearly this step was failing.</p>
        <p>Full disclosure: I’m an ass and this was before I realised that I also had a raspberry pi 3 b with me and
            could have just inserted memory cards here and there to set it up. I did realise this later on which made my
            life much easier. In short, <strong>avoid headless setup</strong> if you’re new to everything like I am.</p>
        <p>I decided to abandon the openwrt part and try and setup the steps myself. To do this I would first have to
            setup communication over raspberry pi zero W’s usb port. This is well documented and is called gadget mode.
            Some references:</p>
        <ol>
            <li>
                <p>If you just want a basic setup without static ip addresses and wifi:</p>
                <ul>
                    <li><a href="https://www.youtube.com/watch?v=xj3MPmJhAPU">Windows</a></li>
                    <li><a
                            href="https://raspberrypi.stackexchange.com/questions/66431/headless-pi-zero-sshaccess-over-usb">Linux</a>
                    </li>
                </ul>
            </li>
            <li>
                <p>A more <a href="https://www.youtube.com/watch?v=MJ084wtjiWM">advanced setup</a> with static iptables
                    and whatnot.</p>
            </li>
        </ol>
        <p>My use case was the second one. But I tried the former first just to get a feel of everything. I used the dd
            command to burn image to my sd card. You can also use rpi-imager but it takes longer for some reason to do
            the same amount of work.</p>
        <p>After burning just follow the steps in the video and you’ll have an rpizero setup for headless mode.</p>
        <p>But WAIT!</p>
        <p>I’d like to point out some snags I faced along the way which are poorly documented.</p>
        <ol>
            <li>When you boot into your pi for the first time and run raspi-config be wary of the order you do things.
                For some reason raspberry pi does not like the fact that we enable ssh server and THEN change the
                hostname and password and reboot it. The ssh file that you created in the /boot directory goes away and
                you can’t ssh anymore. I mean its not magic it makes sense. But try and follow the following order to be
                safe:
                <ol>
                    <li>Insert the sd card with g_ether settings enabled and connect the rpi via usb.</li>
                    <li>Set the wired connection/usb ethernet option to ‘shared among computers’ on your host pc.</li>
                    <li>ssh into the pi via ssh@raspberrypi.local</li>
                    <li>run raspi-config and change the username and password</li>
                    <li>Change timezone and localisation options</li>
                    <li>Set the wireless LAN ssid and password</li>
                    <li>Enable SSH server. Exit raspi-config but dont reboot!</li>
                    <li>cd /boot</li>
                    <li>touch ssh</li>
                    <li>sudo reboot</li>
                </ol>
            </li>
        </ol>
        <p>The last three steps just ensure that ssh will still work across the reboot.</p>
        <ol start="2">
            <li>
                <p>If you want to setup static IP’s the second video is a very good guide but it doesn’t mention some
                    important details. You do need to follow it before reading the rest of this post</p>
                <ul>
                    <li>To setup static ip’s the guy in the video sets up an iptable to route traffic through an
                        interface using its constant name and IP address. I faced a weird problem though. My MAC address
                        at the usb ethernet kept changing everytime I connected. This was truly weird for me as my
                        novice networking knowledge said that mac addresses remain constant and unique. But it is what
                        it is. I found a <a href="https://www.raspberrypi.org/forums/viewtopic.php?t=171791">good
                            post</a> that addressed the issue and solved it for me as well. Hail internet.</li>
                    <li>As this post has revealed and will continue to do so, there are a lot of interfaces and routes
                        that we need to configure to our liking. If you know what you’re doing and are willing to debug,
                        I would suggest disabling dhcp server. This is a service that automatically configures and
                        assigns ip addresses to devices on the network. I wasted a lot of time due to random issues that
                        I could not resolve and didn’t know the cause for.<br />
                        (I also tried disabling avahi-daemon, wpa_supplicant and ifplugd. but only dhcp works
                        fine)<br />
                        To keep a static ip at my usb ports I therefore did the following (along with following the
                        second video)<br />
                        On the pi zero:</li>
                </ul>
                <pre class=" language-backgrouund"><code class="prism :black language-backgrouund">$ sudo systemctl disable dhcpcd  
$ sudo nano /etc/interfaces.d/usb0 # new file with the contents in the snippet below
</code></pre>
                <pre><code>auto usb0
allow-hotplug usb0
iface usb0 inet static
    address 10.42.1.10   # can be any thing you want
    netmask 255.0.0.0    # this might be different for you
    gateway 10.42.1.1    # should be in the same subnet as the ip address
</code></pre>
                <ul>
                    <li>The gateway is the address that will be assigned to the interface at your host pc. The video
                        explains this well. The netmask migh be different than mine. How to find i? First connect your
                        usb without any static ip. Type ifconfig and you’ll see the netmask under your interface name.
                    </li>
                    <li>Similarly you can set static ip’s for wlan0 etc. in the directory. I understand that
                        /etc/network/interfaces is an outdated method and dhcp is better but hey. Whatever floats your
                        boat. DHCP just did’nt work and trust me, I tried a lot.</li>
                </ul>
            </li>
            <li>
                <p>Another weird thing that happens is that whenever I connect two rpi zeros over two usb ports with
                    different ip and MAC address, I can only ping one at a time! If enable connections for both only the
                    one connected first will work. Just something to look out for.</p>
            </li>
        </ol>
        <p>The above steps are not exhaustive. They never are. You WILL have to debug more than this as there are very
            random events that take place in networking apparently. For eg. a simple restart of network-manager doesn’t
            work upon any configuration change but a reboot does the job.</p>
        <p>Make sure to reboot a lot! It solves a lot of doubts</p>
        <p>After the above tinkering I was able to ping over a static ip on the usb connected to the raspberry pi’s and
            also connect to the same over wifi as well. I’ll talk about meshes and batman in the next post.</p>
        <p>Till then. Thanks for reading!</p>
        <p><strong>Update 12/03/2021:</strong></p>
        <ol>
            <li>I found the solution for problem/weird thing number 3 from the above post. It was lack of knowledge on
                my part of course.</li>
        </ol>
        <p>The problem was that once i connected/enabled one interface the other one would not return any ping. The
            thing I did not know was that the route’s host endpoint depends on the netmask you set for that
            interface.<br />
            So say if you have an iface ip address of <code>10.42.2.20</code> and a netmask of <code>255.0.0.0</code>
            then the route will start from a base address of <code>10.0.0.0</code>. You can see this (like I did) by
            typing <code>ip route</code> in the terminal.</p>
        <pre><code>$ ip route
10.0.0.0/24 dev enx000000000009 proto kernel scope link src 10.42.1.1 
10.0.0.0/24 dev enx000000000019 proto kernel scope link src 10.42.2.1 
192.168.29.0/24 dev wlp4s0 proto kernel scope link src 192.168.29.17 metric 600 
</code></pre>
        <p>As you can see I have both interfaces up but I could ping only one because I had a netmask value of 8 which
            resulted in the same base point for both interfaces. All I had to do was change the netmask to
            <code>255.255.255.0</code> and voila!<br />
            After a reboot, I could ssh into both rpis simultaneously. Again, this is because I was changing the subnet
            for both interfaces and the netmask wasnt reflecting that.</p>
        <ol start="2">
            <li>Another cool thing that I noticed after literally days of debugging was that once I disabled dhcp on the
                pi and made the ip static, I could just add the interface on my host pc with the right gateway address
                and netmask and it would connect without any changes to the ip table.(Although I'm entirely convinced
                there's someone sitting behind the laptop conjuring such tricks and making a fool of me. I almost always
                am convinced whenever using Linux :) )</li>
            <li>There was a new prpblem that I faced which I haven’t been able to solve as of yet.f you have the
                raspberry pis connected, don't turn off/reboot the host pc without turning them off from the remote
                terminal first. For some reason, this makes the host pc loose knowledge of the mac addresses of the
                boards and you won't be able to login again.</li>
        </ol>
    </div>
</body>

</html>