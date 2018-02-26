This is a fix I am rolling out to anyone who uses a Debian or Debian derivative and uses a Dell 7577 Series Laptop with a broken iwlwifi driver in it.


Included is a example script of how to fix the infighting interfaces. Usually you will not have ANY warning that your interfaces are literally de-authing one another unless you run parprouted against any two network interfaces on your laptop.


`parprouted -d $ifaceone $ifacetwo`

If you see this:

![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")
![Infighting Wireless and Ethernet Cards Deauthing One Another if you run parprouted on a suspected faulty device](https://github.com/tanc7/Debian-Fix-Infighting-Intel-NICs/blob/master/infightingifaces.png)

Then they are fighting each other. Locking you out of internet access. They basically are trying to shut each other down and bring each other up. There will be NO indication that this is happening. Outside of parprouted. 

It usually shows up as a ARP Request or a Request to shut down another device. 

To fix it, you have to write your own script using mine as a template.

# One. Shutdown all Interfaces on bootup

First kill all interfering services
`killall wpa_supplicant wicd network-manager
rm -rf /var/run/wpa_supplicant/*`

The NICs are located in /sys/class/net each folder is a interface
Just 

`cd /sys/class/net`
`ls >> list-of-interfaces.txt`

Then for each interface tell ifconfig to shut it down.

`ifconfig $iface down`

# Two. Enable the NEEDED PHYSICAL interface after you boot up

That means you only need ONE and no more than that. Are you gonna use ethernet? Or wifi? Pick one right now. Don't bring the other one up or the infighting starts again.

`rfkill unblock all
ifconfig $iface up`


# Three. Enable the NETWORK MANAGER for your interface

Whatever that may be. I use wpa_supplicant and wicd. 

`wpa_supplicant -i $iface -c $config`

if that failed I'd kill it and go

`service wicd start
wicd-curses`

I also added more to clarify how I fixed my KVM Hypervisor's network. I combined parparouted with static routing. This means even if wireless got downed, or my tables got screwed, I still can maintain a network connection because I merged my wireless and ethernet cards together with a proxy arp protocol daemon (thats parprouted).
