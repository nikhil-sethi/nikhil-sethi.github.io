---
layout: post
title:  "Decentralized drone swarms: Part 2 (communication)"
date:   2021-03-17
categories: networking raspberry-pi communication ad-hoc ALFRED BATMAN swarms  
---

<p>You… You’re back?</p>
<p>That seems like a lot of pressure. Oh well.<br /></p>

We’ll discuss something cool today-mesh networking. But before I delve into that, I realised that I hadn’t taken out
time to lay out the problems in what we’re trying to achieve. So let’s start with that for a bit<br />
There are certain key features of any decentralised multi-agent system:<p></p>
<ol>
    <li>It should be <b>decentralised</b>! As the word implies, the control of each member in the swarm is distributed
        and doesn’t depend on just one single member. This also makes a system scalable without too much traffic over
        the communication.</li>
    <li>The communication system should be such that each member can afford to go out or com inside the network at any
        point of time without disrupting it. This is the basic property of an <b>ad-hoc</b> network. It is also called
        self -healing at time.</li>
    <li>The members should be <b>homogenous</b>. Now, this isn’t an absolute requirement in every possible sense to be
        fair. This property was adapted from natural systems as shown in the first post. In such natural systems it is
        highly common for all the agents to possess the same hardware and capabilities. But we can tweak this a little
        to our use case. Strictly speaking the agents must be homogenous in terms of their networking capability and
        capacity but can have different hardware and software at the lower level.</li>
    <li>The <b>whole should be greater that the sum</b> of it’s parts. This means that at the end of the day, the swarm
        must execute a meaningful task <b>with</b> the cooperation of each member and the task should take lesser
        resources than they would if summed individually over each member. This resource can be any unit of importance
        like time, money or energy.</li>
</ol>
<h2 id="so-what-do-we-need-to-solve" style="text-align: left;">So what do we need to solve?</h2>
<p>There are in summary three fields of problems that we need we can collect everything into:</p>
<ol>
    <li>Communication</li>
    <li>Control</li>
    <li>Hardware</li>
</ol>
<p>In the past I have worked to some degree on the hardware problem and I am just start to get my feet wet with the
    first two. I intend to bathe.</p>
<h2 id="brief" style="text-align: left;">Brief</h2>
<p>In this post we’ll do the following:</p>
<ol>
    <li>Setup an adhoc mesh network between two raspberry pi zeros (not technically a mesh)</li>
    <li>Do a range test of this network</li>
    <li>Send and receive data on this network.</li>
</ol>
<p>Onto mesh networking.<br />
    This is obviously a communication problem. From the hardware side it requires that each drone be equipped with an
    appropriate device which can transmit and receive signals in the electromagnetic spectrum. For extremely small,
    short range and low power tasks, wifi is one of the best choices for such systems and that is what I have chosen for
    this project.<br />
    Each UAV will have on board a small computer and a wifi adapter which needs to work in both client(station) and
    server(access point)** mode to transmit and receive signals.<br />
    For this project, I chose the raspberry pi zero W which combines both capabilities into a very small an light PCB. I
    must admit that before starting I did not know if the on board wifi will work for such setup. I had and educated
    guess at best. Spoiler alert! It does.</p>
<p>The software part is where this gets tricky. The following part of the blog post is just resources collected from the
    internet and my own debugging to setup all of it up. We need a protocol or some sort of <b>client server setup which
        supports mesh networking using adhoc networks with raspberry pi</b> **As it turns out there is some nice work
    that has been done by others to implement this. Some notable candidates were:<br />
    5. <a href="https://www.open-mesh.org/projects/batman-adv/wiki">B.A.T.M.A.N Advanced</a><br />
    6. <a href="https://www.irif.fr/~jch/software/babel/">Babel</a><br />
    7. OSLR</p>
<p>According to <a href="https://iopscience.iop.org/article/10.1088/1757-899X/875/1/012046/pdf">this</a> reference, OSLR
    is better in terms of connectivity but poorer than batman in return time delay. I could’nt find many resources of
    babel and support pages for use on raspberry pi’s. There was a lot of support for batman on the internet and many
    posts discussing the process so I decided to go ahead with that. Also, Batman sounds way cooler than the rest.</p>
<p>** Technically in an adhoc network there is no client and server but a distinction was made just for clarity.</p>
<p>There are also a lot of good posts that I followed and are worth mentioning here:</p>
<ol>
    <li><a href="https://github.com/binnes/WiFiMeshRaspberryPi">https://github.com/binnes/WiFiMeshRaspberryPi</a></li>
    <li><a
            href="https://medium.com/swlh/setting-up-an-ad-hoc-mesh-network-with-raspberry-pi-3b-using-batman-adv-1c08ee565165">https://medium.com/swlh/setting-up-an-ad-hoc-mesh-network-with-raspberry-pi-3b-using-batman-adv-1c08ee565165</a>
    </li>
    <li><a
            href="https://medium.com/@tdoll/how-to-setup-a-raspberry-pi-ad-hoc-network-using-batman-adv-on-raspbian-stretch-lite-dce6eb896687">https://medium.com/@tdoll/how-to-setup-a-raspberry-pi-ad-hoc-network-using-batman-adv-on-raspbian-stretch-lite-dce6eb896687</a>
    </li>
    <li><a
            href="https://github.com/suiluj/pi-adhoc-mqtt-cluster/wiki/Batman-Adv-and-Batctl">https://github.com/suiluj/pi-adhoc-mqtt-cluster/wiki/Batman-Adv-and-Batctl</a>
    </li>
    <li><a
            href="https://www.raspberrypi.org/forums/viewtopic.php?t=24615">https://www.raspberrypi.org/forums/viewtopic.php?t=24615</a>
    </li>
    <li><a
            href="https://www.reddit.com/r/darknetplan/comments/68s6jp/how_to_configure_batmanadv_on_the_raspberry_pi_3/">https://www.reddit.com/r/darknetplan/comments/68s6jp/how_to_configure_batmanadv_on_the_raspberry_pi_3/</a>
    </li>
</ol>
<h2 id="setup" style="text-align: left;">Setup</h2>
<p>In summary, with the configuration that we setup earlier on our pi zeros we need do the following on each pi after
    ssh’ing into them via usb.<br />
    Note: Here I am assuming that on your rpi, both usb over ethernet and wifi are working. You will need this for the
    moment to install packages (but not for long).</p>
<bcode>$ sudo apt-get install batctl alfred python3-dev python3-pip git
    $ sudo apt-get remove python2; sudo apt-get autoremove
    $ pip3 install pymavlink setuptools # (pip may take long time to load on rpizero. its normal)
    $ sudo reboot
</bcode>
<p>All the above are just some important packages you’ll need.<br />
    A small note regarding alfred: You might want to clone source repository from github ad compile/cross-compile it on
    the dev/host pc. Depending on the time that whoever(if anyone) is reading this, the debian repository may have been
    updated. At the time of writitng it isn’t. I won’t be covering this for the sake of brevity.<br />
    Now that we have a lot of the things installed, let’s go ahead and setup the ad-hoc network. Note that after doing
    this step, the wifi will no longer work for the internet or ssh purposes. The pi will create it’s own network. So
    make sure that you can ssh into your raspberrypi through the usb cable in a reproducible manner. (Don’t worry. We’ll
    solve the internet problem)<br />
    You can follow the above blog posts for this part and make an average conclusion of steps. In summary you need to do
    the following.<br />
    On the pi</p>
<ol>
    <li>Create a file in the home directory called <i><a href="http://start-batman-adv.sh">start-batman-adv.sh</a></i>.
        File contents:</li>
</ol>
<bcode>#!/bin/bash

    sudo ip link set wlan0 down
    sudo iw wlan0 set type ibss # ibss is another name for adhoc
    sudo ip link set wlan0 up
    sudo iw wlan0 ibss join rpiadhoc 2432 key d:1:&lt;any WEP hex key here&gt;

    modprobe batman-adv
    # Tell batman-adv which interface to use
    sudo batctl if add wlan0

    # Activates the interfaces for batman-adv
    sudo ip link set dev wlan0 up
    sudo ip link set dev bat0 up
</bcode>
<ol start="2">
    <li>Do the same for both pi’s and make sure the key and ssid (rpiadhoc/anything else) is same on both.&nbsp;</li>
</ol>
<ol start="2">&nbsp;<li>Reboot both.<br /></li>
    <li>
        <icode>$ batctl if</icode> Shoulg give something like <icode>wlan0: active</icode>
    </li>
    <li>
        <icode>$ sudo batctl n</icode><br />
        Should give something like.
    </li>
</ol>
<bcode>[B.A.T.M.A.N. adv 2019.4, MainIF/MAC: wlan0/b8:27:eb:e2:a8:61 (bat0/86:50:2f:39:90:62 BATMAN_IV)]
    IF Neighbor last-seen
    wlan0 &lt;other pi's wlan0 mac address&gt; 0.570s
</bcode>
<h2 id="testing" style="text-align: left;">Testing</h2>
<p>And you’re almost done! Your pi can see it’s neighbors in the mesh network. At this point you can run some range
    tests to see the network strength. I did this by connecting both pis to different laptops and moving around the
    house/street. (Get ready for your own neighbors to suspect you doing something “fishy” -.-)</p>
<p></p>
<div class="separator" style="clear: both; text-align: center;"><a
        href="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/multiple_laptop_comms.png"
        imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="1152"
            data-original-width="2048" height="402"
            src="https://raw.githubusercontent.com/nikhil-sethi/nikhil-sethi.github.io/refs/heads/main/_posts/media/multiple_laptop_comms.png"
            width="710" /></a></div><br />
Keep the first pi static and type <icode>$ sudo batctl ping &lt;first pi's wlan0 mac address&gt;</icode> in the second
pi before moving around to see where the ping starts to fail.<br />
The results I got (ping in milliseconds):<p></p><br />

<table>
    <thead>
        <tr>
            <th>Distance</th>
            <th>Obstructions?</th>
            <th>#Pi</th>
            <th>min</th>
            <th>avg</th>
            <th>max</th>
            <th>mdev</th>
            <th>&nbsp; loss %</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>0m</td>
            <td>No</td>
            <td>1</td>
            <td>1.850</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>7.731</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>11.663</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>2.493</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>0%</td>
        </tr>
        <tr>
            <td>0m</td>
            <td>No</td>
            <td>2</td>
            <td>1.744</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>7.857</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>12.250</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>2.350</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>0%</td>
        </tr>
        <tr>
            <td>10m</td>
            <td>No</td>
            <td>1</td>
            <td>1.866</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>8.733</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>12.907</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>1.973</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>0%</td>
        </tr>
        <tr>
            <td>10m</td>
            <td>No</td>
            <td>2</td>
            <td>1.837</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>7.928</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>12.398</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>2.356</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>0%</td>
        </tr>
        <tr>
            <td>20m</td>
            <td>Yes</td>
            <td>2</td>
            <td>2.537</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>11.788</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>26.948</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>5.629</td>
            <td><span>&nbsp;&nbsp; &nbsp;</span>12%</td>
        </tr>
    </tbody>
</table>
<p>The pi is very sensitive to obstructions in its immediate vicinity. So, if the pi is lying down somewhere for
    instance, the ping fails. But hang it loose and it gives good results. We can keep this in mind while placing the pi
    on the quadcopter (of course. don’t hang it loose).<br />
    I expect to get 30meters without any obstructions which is the advertised value.</p>
<h2 id="whats-next" style="text-align: left;">What’s next?</h2>
<p>We’ve established a network and tested the range by sending arbitrary data. Let’s try and send some meaningful data
    now which is our eventual goal. There is a dedicated service named alfred (of course…) made by the good people at
    open-mesh. Assuming you already downloaded and compiled it, run:<br />
    On both pis: <code>$ sudo alfred -i bat0 -m -p 1 &gt;&gt;/dev/null &amp;</code><br />
    The <icode>-i</icode> is for the the interface name, <icode>-p</icode> is the refresh rate interval in seconds,
    <icode>-m</icode> is the option to make the alfred server primary in nature. Go through the alfred documentation to
    understand what this means. The <icode>&gt;&gt; /dev/null</icode> is just something to redirect all output to an
    empty space and <icode>&amp;</icode> is to run the service in the background</p>
<p>On pi number 1: (send)</p>
<bcode>~$ cat /etc/hostname | sudo alfred -s 64 # sudo rights are important!
</bcode>
<p>This command just sends the hostname of the sender over the network</p>
<p>On pi number 2: (receive)</p>
<pre><bcode>$ sudo -i 
/$ alfred -r 64
{ "62:6b:d1:f7:14:0d", "rpizero1\x0a" },
</bcode></pre>
<p>Wohoo. That’s data! The mac address above should match with the bat0 mac address on pi number 1. and the data under
    quotes is what you sent from the sender.</p>
<h2 id="some-problems" style="text-align: left;">Some problems</h2>
<p>If you’re getting the error:</p>
<ol>
    <li>
        <icode>read from unix socket failed: No such file or directory</icode>. It means that something is wrong with
        your send command
    </li>
    <li>
        <icode>can't connect to unix socket: No such file or directory</icode> <b>OR</b>
        <icode>can't connect to unix socket: Connection refused</icode> . It means that your alfred server didn’t start
        for some reason. Check with <icode>ps aux | grep alfred</icode> to see if there is a valid process running.
    </li>
    <li>
        <icode>can't connect to unix socket: Permission denied</icode>. It means something is obviously wrong with your
        permissions. Try logging into a terminal as root by <icode>sudo -i</icode> and then try sending and receiving
        data.
    </li>
    <li>The alfred server reads data from a unix socket located as /var/run/alfred.sock. Sometimes this file may not be
        created by the default installation for some reason. Just do <icode>touch /var/run/alfred.sock</icode> and
        you’re set.</li>
</ol>
<p>There will probably be more errors which can be debugged and will in all likelihood, solvable. Up to you. That’s all
    from my side for now.<br />
    In the next post I’ll move onto using such an architecture through python and making a program which connects
    ardupilot’s SITL, BATMAN and ALFRED to publish vehicle states over this network.</p>
<p>B Y E</p>