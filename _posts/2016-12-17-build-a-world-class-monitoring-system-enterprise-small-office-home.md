---
layout:     post
title:      Continuous Monitoring - Build a World-Class Monitoring System for Enterprise, Small Office or Home
date:       2016-12-17 00:00:00
time:   5
summary:    This paper outlines guidance for network visibility, threat intelligence implementation and methods to reduce analyst alert fatigue.

categories: suricata elasticsearch logstash continuous monitoring intrusion detection system
products:
 - top-level: Home System Setup
   arbitrary: Hardware
   nested-products:
    - nested: Network TAP, [Dualcomm-DCGS-1000Base-T](https://www.amazon.com/Dualcomm-DCGS-2005L-1000Base-T-Gigabit-Network/dp/B004EWVFAY/)
      sub-arbitrary: A gigabit network tap is not required for most home networks.

author: austin_taylor
author_github: austin-taylor
---


![HTTPMAP]({{ site.base }}/images/http_map.png){: class="center-image" height="500px"}

<h2>Overview</h2>
On December 15th, 2016 [SANS published my gold paper](https://www.sans.org/reading-room/whitepapers/detection/continuous-monitoring-build-world-class-monitoring-system-enterprise-small-office-home-37477) which included recommendations for
 Intrusion Detection System (IDS) setup and tips for efficient data collection, sensor placement, identification of critical infrastructure along with network and metric visualization.

 Based on feedback requesting step-by-step implementation, this blog post serves as a supplement to the gold paper to implement continuous monitoring in your home. This post will also include specific hardware recommendations and direct links for software download.

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
* It is possible to listen on the same interface that you're management port is on (the port with an IP address), but it is best to have a dedicated interface.
* Per [SELKS Github](https://github.com/StamusNetworks/SELKS) the minimal configuration for production usage is 2 cores and 4 Gb of memory. As Suricata and Elastisearch are multithreaded, the more cores you have the better it is. Regarding memory, the more traffic to monitor you have, the more getting some extra memory will be interesting. See [Running SELKS in production](https://github.com/StamusNetworks/SELKS/wiki/Running-SELKS-in-production) page for more info.

Software
-----

Next, we'll need to download SELKS.

1. Download the [SELKS](https://www.stamus-networks.com/open-source/) ISO. This will be installed on the server.


Our Goal
---
Gain network visibility into an enterprise, small office, or home network.

Here is an example of a network topology. The topology below may be more relevant toward a small office, but we'll use segments to emulate a home network. Many home networks may not have a switch or firewall connected (not be a bad idea to get one though!)

![NETWORK_TOPOLOGY]({{ site.base }}/images/all_together_labeled.png)

Setup
---
1. Create a bootable USB Thumbdrive with the SELKS ISO. If needed, assistance available [here](http://www.howtogeek.com/191054/how-to-create-bootable-usb-drives-and-sd-cards-for-every-operating-system/)
2. Insert thumbdrive into server and boot. **May need to set server to boot from USB in BIOS.**
   * If all goes well, you should see SELKS boot menu. Pressing enter will lead you to the graphical interface.
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

Lastly, you'll want to follow the tuning considerations on the [SELKS wiki page](https://github.com/StamusNetworks/SELKS/wiki).

Recommendations on the page include:

    1. Initial Setup
    2. Tuning and Maintenance
    3. Data and Logs
    4. Troubleshooting and Getting Help

If you don't tune Elasticsearch or Suricata, the stack will eventually fail. Your server **MUST** be configured or the availability will not be reliable.

Hope you found this guide helpful and if you have any questions, please post them in the comments section below.







