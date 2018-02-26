# This is a fix I am rolling out to anyone who uses a Debian or Debian derivative and uses a Dell 7577 Series Laptop with a broken iwlwifi driver in it.
```
Chang Tan
changtan@listerunlimited.com
Lister Unlimited Cybersecurity Solutions, LLC.
```

Lets be very clear. I am NOT Intel. I am NOT Dell. I am not part of Ubuntu or Debian. I do not represent them in any way. But I am very mad after realizing that my network drivers are faulty and I need to use wireless dongles, and thousands of lines python and bash scripts before I figured out what happened.

Now I don't know whose ass am I supposed to kick. But consider trying this fix out first (follow my instructions to make a script that manually manages the interfaces one by one), and then go notify whoever dunnit wrong (seems to be one of the network card drivers) i'm gonna go kick his ass for wasting my time.

# Installation

You can either play around and edit with my scripts or execute them (if the settings are YOUR NETWORK's which is nsot) AFTER you edit them.

To make them executable,

```
git clone https://github.com/tanc7/Debian-Fix-Infighting-Intel-NICs
cd Debian-Fix-Infighting-Intel-NICs
chmod 777 *.sh
./some-fix.sh
```

This forces a mandatory shutdown and restart network interfaces. And itll finally work, hopefully.

In my laptop the faulty card is this one
```
        *-pci:3
             description: PCI bridge
             product: Sunrise Point-H PCI Express Root Port #6
             vendor: Intel Corporation
             physical id: 1c.5
             bus info: pci@0000:00:1c.5
             version: f1
             width: 32 bits
             clock: 33MHz
             capabilities: pci pciexpress msi pm normal_decode bus_master cap_list
             configuration: driver=pcieport
             resources: irq:125 memory:dd100000-dd1fffff
           *-network
                description: Wireless interface
                product: Intel Corporation
                vendor: Intel Corporation
                physical id: 0
                **bus info: pci@0000:3c:00.0**
                **logical name: wlp60s0**
                version: 78
                serial: 5c:e7:0f:44:08:2d
                width: 64 bits
                clock: 33MHz
                capabilities: pm msi pciexpress bus_master cap_list ethernet physical wireless
                **configuration: broadcast=yes driver=iwlwifi driverversion=4.9.0-6-amd64 firmware=22.361476.0 ip=10.0.1.50 latency=0 link=yes multicast=yes wireless=IEEE 802.11**
                resources: irq:131 memory:dd100000-dd101fff
```

# How do confirm if you have a faulty card too

Included is a example script of how to fix the infighting interfaces. Usually you will not have ANY warning that your interfaces are literally de-authing one another unless you run parprouted against any two network interfaces on your laptop.


```parprouted -d $ifaceone $ifacetwo```

If you see this:

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")
![Infighting Wireless and Ethernet Cards Deauthing One Another if you run parprouted on a suspected faulty device](https://github.com/tanc7/Debian-Fix-Infighting-Intel-NICs/blob/master/infightingifaces.png)

Then they are fighting each other. Locking you out of internet access. They basically are trying to shut each other down and bring each other up. There will be NO indication that this is happening. Outside of parprouted. 

It usually shows up as a ARP Request or a Request to shut down another device. 

To fix it, you have to write your own script using mine as a template.

# One. Shutdown all Interfaces on bootup

First kill all interfering services
```
killall wpa_supplicant wicd network-manager
rm -rf /var/run/wpa_supplicant/*
```
The NICs are located in /sys/class/net each folder is a interface
Just 
```
cd /sys/class/net
ls >> list-of-interfaces.txt
```
Then for each interface tell ifconfig to shut it down.

```ifconfig $iface down```

# Two. Enable the NEEDED PHYSICAL interface after you boot up

That means you only need ONE and no more than that. Are you gonna use ethernet? Or wifi? Pick one right now. Don't bring the other one up or the infighting starts again.
```
rfkill unblock all
ifconfig $iface up
```

# Three. Enable the NETWORK MANAGER for your interface

Whatever that may be. I use wpa_supplicant and wicd. 

```wpa_supplicant -i $iface -c $config```

if that failed I'd kill it and go
```
service wicd start
wicd-curses
```
I also added more to clarify how I fixed my KVM Hypervisor's network. I combined parparouted with static routing. This means even if wireless got downed, or my tables got screwed, I still can maintain a network connection because I merged my wireless and ethernet cards together with a proxy arp protocol daemon (thats parprouted).

# Four. Add it as a cronjob until someone rolls out a real fix

Add these two to the top of the script
```
#!/bin/sh
SHELL=/bin/bash
```
And add these to your crontab first. crontab -e
```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/etc/init.d"
SHELL=/bin/bash
initpath=/etc/init.d
binpath=/usr/local/bin
```
Then ```cp -r script.sh``` to /usr/local/bin and go crontab -e again.

copy this to the bottom of crontab,save and exit and reboot.

```@reboot /bin/sh $binpath/script.sh```

