---
layout:     post
title:      Detect Beaconing with Flare, ElasticCloud, and Intrusion Detection Systems
date:       2017-06-10 00:00:00
time:   5
summary:    Detect Periodic Behavior (Beaconing) in an environment already running an IDS and ElasticCloud

categories: detect beaconing intrusion detection system command control flare

author: austin_taylor
author_github: austin-taylor
---


![MINUTE_BEACONS]({{ site.base }}/images/minute_beacon_1.png){: class="center-image" height="200px"}

<h2>Overview</h2>

After some variants of malware infect a computer, it can attempt to check in with its command and control server in periodic time intervals. This activity is referred to as "beaconing".

Identifying network beaconing can be critical to preventing an adversary from taking action on an objective. In the Lockheed Martin Killchain, Beaconing would be in the step prior to an Advanced Persistent Threat (APT) gaining access.

![KILLCHAIN]({{ site.base }}/images/killchain_lockheed_martin.png){: class="center-image" height="200px"}


But it can be difficult to detect this activity. The beaconing can take place at any time or frequency—from once every couple of seconds to once a week (or possibly even longer if you are dealing with an advanced adversary).

Getting Started
---



Hardware
-----

First we'll need to get a few pieces of hardware.

| Hardware | Description | Cost |
| ------------- |:-------------:| -----:|
| TAP or Switch     | TAP or Switch that supports spanning [Dualcomm-DCGS-1000Base-T](https://www.amazon.com/Dualcomm-DCGS-2005L-1000Base-T-Gigabit-Network/dp/B004EWVFAY/)  | $179 |
| Server     | [Server](https://www.amazon.com/SHUTTLE-LGA1151-Skylake-Barebone-SZ170R8/dp/B01C87CQEK/) will run [SELKS](https://www.stamus-networks.com/open-source/) -- See below for minimum requirements|   $0-2000+ |
| Dual NIC Card | Server should be equipped with two ports. One for management and another for sniffing. NICs available [here](https://www.amazon.com/Intel-1000-Dual-Server-Adapter/dp/B000BMZHX2) | $46 |
| Management Switch | [Network switch](https://www.amazon.com/Netgear-ProSafe-48-Port-Gigabit-GS748TNA/dp/B00062WV9U) to separate your management network. |    $10-$400+ |
| ISP Provided Router | This is the DSL/Cable modem provided by your internet provider | Monthly Bill |


A few caveats:
---
* All products with links are personal preference. I'm sharing the setup of my network, but feel free to use replacements.
* There are many ways to monitor network traffic. Network TAPs are the cleanest way to do it. The recommended TAP above serves as a gigabit switch and can be powered by a USB. Choose a TAP that suits you. In many cases, 100Mbps is okay, but may suffer from packet loss if the network operates at greater speeds.
* It is possible to listen on the same interface that your management port is on (the port with an IP address), but it is best to have a dedicated interface.
* Per [SELKS Github](https://github.com/StamusNetworks/SELKS) the minimal configuration for production usage is 2 cores and 4 Gb of memory. As Suricata and Elastisearch are multithreaded, the more cores you have the better it is. Memory is used by ElasticSearch for indexing network traffic. High traffic networks will require more memory. I have 32GB on my sensor, of which 12-19GB is consistently in use. See [Running SELKS in production](https://github.com/StamusNetworks/SELKS/wiki/Running-SELKS-in-production) page for more info.

Software
-----

Next, we'll need to download SELKS.

1. Download the [SELKS](https://www.stamus-networks.com/open-source/) ISO. This will be installed on the server.


Our Goal
---
Gain network visibility into an enterprise, small office, or home network.

Here is an example of a network topology. The topology below may be more relevant toward a small office, but we'll use segments to emulate a home network. Many home networks may not have a switch or firewall connected (not a bad idea to get one though!)

![NETWORK_TOPOLOGY]({{ site.base }}/images/sensor_topology_labeled_full_shadow.png)

Setup
---
1. Create a bootable USB Thumbdrive with the SELKS ISO. If needed, assistance available [here](http://www.howtogeek.com/191054/how-to-create-bootable-usb-drives-and-sd-cards-for-every-operating-system/)
2. Insert thumbdrive into server and boot. **May need to set server to boot from USB in BIOS.**
   * If all goes well, you should see SELKS boot menu. Pressing enter will lead you to the graphical interface.

       **Users booting from a thumbdrive may need to follow these additional steps.**

       1. At language prompt, Press ALT-f2
       2. Type _mkdir /cdrom_
       3. Type _mount /dev/sdb1 /cdrom_
            * Your parition name may not be sdb1. Use fdisk -l to list available partitions
       4. Press Alt-F1 to return to the installation process and continue.


   * Default username and password is selks-user/selks-user and root is ``StamusNetworks``
     * More information available at [SciriusUsage](https://github.com/StamusNetworks/scirius#usage)
3. Login to server and assign a static IP address to eth1. For example, if your network uses the 192.168.1.0/24 range you can assign 192.168.1.250 to interface eth1 on your server.
4. Install your network TAP inline with your ISP provided router.
   * If using the recommended TAP you can use the following configuration:

    | Port | Connected-To | Description |
    | ------------- |:-------------:| -----:|
    | 1 | ISP Router | Passes all home traffic through to router |
    | 2 | Switch or Wireless Router | Plugin your switch or wireless router. If you have multiple wireless access points, plug them into a switch, and plug the switch into port 2. |
    | 5 | Server | Plug into the "sniffing port" on your server. Eth0 is to set sniff by default |

   * Your server should now be collecting network traffic!

5. Login to your server via a web browser. https://server.assigned.ip.address

Tuning
---







