---
layout:     post
title:      Detect Beaconing with Flare, Elastic Stack, and Intrusion Detection Systems
date:       2017-06-10 00:00:00
time:   5
summary:    Detect Periodic Behavior (Beaconing) in an environment already running an IDS and Elastic Stack (formerly ELK)

categories: detect beaconing intrusion detection system command control flare elastic stack

author: austin_taylor
author_github: austin-taylor
---


![MINUTE_BEACONS]({{ site.base }}/images/minute_beacon_1.png){: class="center-image" height="200px"}

<h2>Overview</h2>

After some variants of malware infect a computer, it can attempt to check in with its command and control (C2) server in periodic time intervals. This activity is referred to as "beaconing".

<img style="float: left;height:500px;" src="/images/killchain_lockheed_martin.png">

Identifying network beaconing can be critical to preventing an adversary from taking action on objective. In the Lockheed Martin Killchain, Beaconing would occur prior to an adversary or Advanced Persistent Threat (APT) gaining access.

However, it can be difficult to detect this activity. Beaconing can occur at any time and the frequency can vary. Additionally, network communication does not have perfect intervals or the malware may try to add "jitter" to prevent showing up for someone looking at hard intervals (Example: Every 30 seconds).

There are legitimate uses for periodic communication such as time synchronization and software updates.

<a href="https://github.com/austin-taylor/flare">Flare</a> is a free open-source cyber analytic framework intended to make identifying malicious behavior, such as C2 beaconing in networks, as simple as possible.

This post will use Flare to identify beaconing inside of a network.

Getting Started
---

Before we start, you will need to have the following available.

| Name | Description |
| ------------- |:-------------:|
| Python 2.7     | [Python](https://www.python.org/downloads/) - Required on client workstation |
| IDS | [Suricata](https://www.suricata-ids.org) or [Snort](https://www.snort.org/) - Popular Intrusion Detection Systems |
| Flare | A [python framework](https://github.com/austin-taylor/flare) used for network analysis |
| Elastic Stack | Entire Stack not required, but need [Elastic Search](https://www.elastic.co/products) with flow data |

<hr>
This post assumes you have a network monitoring system and are ingesting the output into Elastic Stack (formerly ELK). If you are interested in getting started with network monitoring, visit my post on <a href="http://www.austintaylor.io/suricata/elasticsearch/logstash/continuous/monitoring/intrusion/detection/system/2016/12/17/build-a-world-class-monitoring-system-enterprise-small-office-home/"> Building a Monitoring System for Enterprise, Small Office or Home Networks.</a>

For this walk-through, we'll be using <a href="https://github.com/StamusNetworks/SELKS">SELKS 4.0</a>, but Flare is modular by design and will work with <a href="https://securityonion.net/">Security Onion</a> or any any system running Snort, Bro, Suricata. The flow data inside Elastic Stack (ES) is what Flare uses to identify beacons.

Additionally, you will need to have <a href="https://github.com/austin-taylor/flare">Flare</a> installed (instructions on github).

<hr>

Build a Tunnel
--------------

Depending on how you have elasticsearch configured, you may need to build an SSH tunnel to allow your computer to communicate with your elasticsearch node.

For example, if your computers IP address is 192.168.1.150 and your elasticsearch node is at 192.168.1.2, you could open port 9200 on your local computer by running:

{% highlight awk %}
ssh -NfL 9200:localhost:9200 selks-user@192.168.1.2
{% endhighlight %}

Once connected you can verify your connection by running in a terminal

{% highlight awk %}
curl localhost:9200
{% endhighlight %}

You should receive the elasticsearch greeting.

<img style="height:200px;" src="/images/verify_connection.png">

Once connectivity is verified, it's time to run Flare.

<hr>
<center><img style="height:150px;" src="/images/flare.png"></center>

Configuring Flare
-----------------

After you've installed Flare, a binary file called *flare_beacon* should be available in your path. This file takes arguments or configuration files.

In the configs directory, there are .ini files. Included is a selks4.ini, which is pre-configured for SELKS 4.0. Be sure to tune the settings for your environment.

 I've added comments to explain each field.

{% highlight awk %}
[beacon]
es_host=localhost           # IP address of ES Host, which we forwarded to localhost
es_index=logstash-flow-*    # ES index
es_port=9200                # Logstash port (we forwarded earlier)
es_timeout=480              # Timeout limit for elasticsearch retrieval
min_occur=50                # Minimum of 50 network occurrences to appear in traffic
min_interval=30             # Minimum interval of 30 seconds per beacon
min_percent=30              # Beacons must represent 30% of network traffic per dyad
window=3                    # Accounts for jitter... For example, if 60 second beacons
                            # occurred at 58 seconds or 62 seconds, a window of 3 would
                            # factor in that traffic.
threads=8                   # Use 8 threads to process (Should be configured)
period=24                   # Retrieve all flows for the last 24 hours.
kibana_version=5            # Your Kibana version. Currently works with 4 and 5
verbose=True                # Display output while running script


#Set these fields to your ES flow fields
field_source_ip=src_ip
field_destination_ip=dest_ip
field_destination_port=dest_port
field_timestamp=@timestamp
field_flow_bytes_toserver=bytes_toserver
field_flow_id=flow_id

#set to false if you are not using default Suricata output.
suricata_defaults = true
{% endhighlight %}

Think of min_occur, min_interval, and min_percent as adjustable knobs. The more you crank the knobs up, the fewer results will output.

_More information is available by typing_ **flare_beacon -h**

## Find Beacons

Now that we have flare configured and installed we are now ready to find beacons on our network.

_For this walk-through I generated a request for HTTP traffic every 60 seconds using this command:_

{% highlight awk %}
watch -n 60 curl http://www.huntoperator.com
{% endhighlight %}

In this scenario, we'll consider our infected endpoint beaconing back to a C2 server.

**Infected IP**: 192.168.0.53 --> **C2 Server**: 160.153.76.129

Let's find it using Flare!

{% highlight awk %}
flare_beacon --group --whois --focus_outbound -c configs/selks4.ini -html beacons.html (or -csv beacons.csv)
{% endhighlight %}

If everything worked, you should see output similar to the following:
<img style="height:200px;" src="/images/run_flare_selks_full.png">

**Caution:** Flare will use the resources specified in your config file and if not configured correctly could freeze your computer. Computational resources are proportionate to the size of your network. _The bigger the network, the more resources are required to compute._

Here is a screenshot of resources on my computer during flare runtime.
<img src="/images/cores_and_memory_h.png">

For my small home network with about 100 nodes, it took about 1 minute to process beacons for a 24 hour period.


Understanding the Options
-------------------------
* **--group**: This will group the results making it visually easier to identify anomalies.
* **--whois**: Enriches IP addresses with WHOIS information through ASN Lookups.
* **--focus_outbound**: Filters out multicast, private and broadcast addresses from destination IPs
* **-c**: Config for flare to use
* **-html**: Outputs results to an HTML file
* **-csv**: Outputs results to CSV file (loses ability to group)

Analyzing Results
-----------------

Now for the fun (and hopefully not scary) part -- You get to analyze the results.

If you used the switches above, your output should look like:

<img src="/images/flare_htmloutput_highlight.png">

Among other traffic, Flare has identified our infected IP communicating to a C2 server. How could you have identified this type of behavior without guilty knowledge?

First, it is important to understand the output fields.


Output Fields
-------------
* **bytes_toserver**: Total sum of bytes sent from IP address to Server
* **dest_degree**: Amount of source IP addresses that communicate to the same destination
* **occurrences**: Number of network occurrences between dyads identified as beaconing.
* **percent**: Percent of traffic between dyads considered beaconing.
* **interval**: Intervals between each beacon in seconds

Flare's output shows 192.168.0.53 communicating to 160.153.76.129 in ~61 second intervals over port 80. Of all the communication between the nodes **97%** has periodic communication. The results also indicate 2 additional internal nodes communicate to the same destination IP (In this case, it was me testing connectivity on multiple hosts.)

You'll also notice the same host performing DNS lookups through Google DNS with the same intervals -- this is because our curl command is looking up the hostname first and then making an HTTP request through curl. Awesome!

Filtering out well-known services such as Google & Amazon can help to narrow your result set. You can then look for communication with a low destination degree and a high percentage of beaconing.

Larger networks will have a lot of activity, so you're probably better off to export your results to csv where you can harness the full power of Excel.

<img src="/images/flare_csvoutput_highlight.png">

Investigating Results
---------------------

After you have created a small subset of interesting traffic to investigate, open Kibana to visualize your findings.

If you're using SELKS, open the SN FLOW dashboard and type in the dest_ip as a filter

<img src="/images/minute_beacon_full2.png">

Drill into the time series, by selecting portions and Kibana will automatically adjust the time interval.

<img src="/images/beacon_hover_highlight.png">

You can now visually see communication occurring in ~60 second intervals from our infected host communicating to a C2 server.

<img src="/images/minute_beacon.png">


Time to pivot to the HTTP Dashboard to get some more information.

Apply the same dest_ip filter which you applied to the Flow dashboard.

    dest_ip:160.153.76.129

<img src="/images/view_http_traffic.png">

<img src="/images/pivot_to_eve.png">

The HTTP Dashboard has provided additional context to our traffic. We now know a system hosting Mac OS X is communicating to a site called www.huntoperator.com

At this point, we can pivot to [EveBox](https://github.com/jasonish/evebox) to view the network traffic across multiple indices by clicking correlate flow.

<img src="/images/evebox_flowid.png">

It appears our C2 beacon has also triggered an alert.

<img src="/images/alert_info.png">

Digging into the allows us to view the payload which triggered the alert and confirms our infected host is making GET request to a C2 node using curl.

For bonus points, we can dig into the profile of the Destination IP to observe it's pattern of life.

<img src="/images/evebox_beacon.png">

EveBox does a great job to help analyst characterize host communication.

Bringing It All Together
------------------------

Using Flare, you identified periodic communication on your network. After narrowing your results, you investigated the interaction and determined an end point using Mac OS X was attempting communication over HTTP to huntoperator.com every 60 seconds and over 24 hour period sent 1.23MB and received 12.70MB from a C2 Server located in Scottsdale, Arizona.

<hr>

_Special thank you to **Jonathan Burkert** and **Justin Henderson** for their contributions to Flare's beaconing detection capability_

If you found this post useful, please share with friends, and I hope you enjoy using Flare!
