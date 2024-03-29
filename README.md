# To Do
- Clean up this repo. It's difficult to follow.
- Add illustrations.

# INTRODUCTION

This repo is documentation on wireless access points. With the goal of clearly explaining them, this document will define them and demonstrate their different types. It will also show all steps necessary to turn a Debian-based Linux distribution into a simple wireless access point.

These are the requirements:

- A PC with both a wireless and an Ethernet interface
- An internet connection

This guide configures a Raspberry Pi on pure Debian 11. However, you may use any PC as long as it satisfies the above requirements. <br></br>

Furthermore, you should have an understanding of subnetting and the OSI model for a deeper comprehension of this topic. <br></br>

# WIRELESS ACCESS POINT DEFINITION

## DEFINITION

A <b><u>wireless AP</u></b> is any device that allows another device a data-link connection to some network via WiFi.
<br></br>
An example would be your wireless SOHO router. It may be the case that you are connected to it with WiFi to view this page, and simultaneously your phone is connected to the same WiFi.
<br></br>

## IMPLICATIONS

Although the definition of wireless APs may seem straight-forward, there is ambiguity in the definition: what is the medium through which the subnet is being connected? If the only possible medium is WiFi and devices on that subnet can connect to devices on another subnet, the AP is a <b>wireless router</b>. On the other hand, if there exists another medium (for example, Ethernet) on the subnet through which devices can communicate, the AP is a <b>wireless bridge</b>. This is an extremely important distinction to make, as it will determine how your AP will connect clients to network resources.
<br></br>

# WIRELESS ROUTERS

In the context of the OSI model, wireless APs are by definition on layer 2. However, routers are layer 3 devices. This begs the question: are wireless routers really APs? 
<br></br>
Although a wireless router is unmistakably a layer 3 device, it is very similar <i>in concept</i> to an Ethernet router and switch connected together; namely, there is a layer 2 connection in between. In the Ethernet network, all nodes in the subnet connect via the switch. There is a router on this network, but this router does not necessarily connect to an outside subnet. Similarly in a WiFi network, the devices connect to the network via the AP, and the AP allows access to outside resources with its two interfaces on different subnets. However, another subnet is not a requirement in this scenario.
<br></br>

# WIRELESS BRIDGES

Unlike wireless routers, wireless bridges are not layer-3-capable. Rather than being a device to connect two networks of different mediums, bridges allow both to exist simultaneously in a single network. This means that it is possible for wired and wireless devices to be on the same subnet. In this sense, wireless bridges are switches that "pretend" WiFi and Ethernet are the same thing.

When bridging two interfaces together, there will likely be a third interface that needs to be configured. Like a loopback interface, this interface does not physically exist. However, it will act as the bridge between the two interfaces.
<br></br>

# WIRELESS REPEATERS?
It is worth noting that wireless repeaters are NOT wireless access points since repeaters are layer 1 devices in nature. Their purpose is only to extend the signal of wireless devices further; they only aid in physically connecting nodes, not in defining methods of communication.
<br></br>

# EXAMPLES
 
As mentioned previously, if your SOHO router has WiFi capabilities, it is a wireless AP. However, it may be surprising to some that this device is a <i>wireless bridge</i>. To illustrate, consider this scenario:
- Your laptop is connected to your WiFi 
- Your desktop is directly connected to your network through Ethernet on one of your router's ports
- You have no ISP, or your router is disconnected from your ISP for any reason

In this scenario, the laptop would still be able to communicate with the desktop even though the two are on different mediums. Therefore, the router is a wireless bridge. 
<br></br>
So if the router is a wireless access point, why is it called a "wireless router"? This confusing term references the fact that the device has both WiFi capabilities and the fact that the device connects your home network to the internet on a layer 3 level. In this specific case, the two facts are not relevant to each other. In truth, a more accurate name would be "all-in-one", especially considering that SOHO routers are also modems, switches, and firewalls. Most SOHO routers have Ethernet options labeled LAN and one labeled WAN. The LAN section contains all your wired and wireless devices. The WAN port is a separate, isolated part of the SOHO router.
<br></br>
<br></br>
Nearly every wireless AP is a wireless bridge. They are often the most convenient when setting up WiFi: no routing implies less overhead when connecting to network resources. As a result, there are very few examples of real wireless routers. However, it may be the case that you want a simple but effective implementation to isolate wireless traffic from the main network. In this case, a wireless router would be ideal. 
<br></br>

# CONFIGURING WIRELESS ACCESS POINTS
Please see the ```wireless_bridge``` and ```wireless_router``` directories for template configuration files. It is recommended that you clone this repo and copy the files to their destination.

## WIRELESS ROUTER CONFIG
The following apt packages must be installed:
- hostapd
- bridge-utils
- wireless-tools
- dnsmasq (or other DNS & DHCP daemon)

This guide assumes that there is neither a DNS nor a DHCP server on the wireless network. Hence, Dnsmasq will be used for both. If you prefer another daemon for each, you can use those instead. Furthermore, this guide will use ```ufw``` to set up NAT to avoid changing your SOHO router's routing table since routing is not a focus of this guide.
<br></br>

The first step is to configure the Ethernet and WiFi interfaces. Edit the file ```/etc/network/interfaces```. Assuming the Ethernet network already has a DHCP server on it, ```eth0``` will be given an IP address, subnet mask, default gateway, and DNS server. 
<br></br>
Next, edit these files:
- /etc/default/hostapd
- /etc/hostapd/hostapd.conf

Unmask and enable hostapd:

```
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
```
Hostapd is a daemon that provides wireless routing/bridging utilities. 

Lastly, edit the following files:
- ```/etc/sysctl.conf```
- ```/etc/dnsmasq.conf```
- ```/etc/ufw/sysctl.conf```
- ```/etc/ufw/before.rules```
- ```/etc/default/ufw```

and add the following rules to the firewall:
```
sudo ufw allow in on wlan0 to any
sudo ufw allow from <NAT outside interface's subnet ID/subnet mask> to any
```

If you want to do port forwarding:
```
sudo ufw allow from any to [destination address] port [port number] proto [tcp/udp]
```

Reboot the device to restart all daemons. You should see a new SSID in your list of wireless networks.

## WIRELESS BRIDGE CONFIG
The following apt packages must be installed:
- hostapd
- wireless-tools
- bridge-utils

The first step is to configure the bridge. Edit ```/etc/networking/interfaces```. Note that there is a new ```br0``` interface in the template config.
<br></br>
Next, edit ```/etc/default/hostapd``` and ```/etc/hostapd/hostapd.conf```. Enable hostapd with the following commands:
```
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
```
When done, reboot to apply the bridge and restart hostapd. 

# CONCLUSION
This document discussed the definition of wireless access point. A wireless access point is defined as any device that allows a layer 2 connection to a network using WiFi technology. The document acknowledged that there are two types of wireless APs: routers and bridges. Wireless routers connect all wireless devices into an isolated subnet, whereas wireless bridges allow devices of two different mediums to connect to the same subnet. Lastly, the document gave a brief guide on how to set up a Debain 11 access point.
