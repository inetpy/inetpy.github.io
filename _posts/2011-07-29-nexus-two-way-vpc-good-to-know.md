---
title: 'Nexus Two Way vPC Good to Know'
author: yandy
layout: post
permalink: /2011/07/nexus-two-way-vpc-good-to-know
image: vpc_001.jpg
twitter_image: vpc_001.jpg
description: Gotcha for deploying two way VPC
summary: Quick overview of topology, two Nexus 7000s and two Nexus 5020s each with two 10Gig connections to each other. In other words, there&#8217;s a total of 40Gig between each switch, and they&#8217;re all in one vPC, like so.
dsq_thread_id:
  - 1238231707
categories:
  - Networking
tags:
  - nexus
  - nexus-5000
  - nexus-7000
  - nexus-two-way-vpc
  - nexus-vpc
  - two-way-vpc
  - vpc
  - mec
  - multi-chassis-link-aggregation
---
Quick overview of topology, two Nexus 7000s and two Nexus 5020s each with two 10Gig connections to each other. In other words, there&#8217;s a total of 40Gig between each switch, and they&#8217;re all in one vPC, like so.

**Diagram**  
<img id="diagram" src="{{ site.url }}/assets/images/Two_Way_vPC.png" alt="Diagram" title="" />
<!--more-->

# The Issue {#theissue}

On of the issues we ran into was that, although the vPC showed as **up**, there was no traffic passing through the trunks. Here&#8217;s the config as it was before we fixed it;

**N7K Config**

<pre lang="plain">interface Ethernet1/17
  description Connect to N5K-1 Eth1/37
  switchport
  switchport mode trunk
  switchport trunk allowed vlan 2-17,28-32,200-207
  spanning-tree port type network
  spanning-tree guard root
  channel-group 10 mode active
  no shutdown

interface Ethernet1/18
  description Connect to N5K-2 Eth1/37
  switchport
  switchport mode trunk
  switchport trunk allowed vlan 2-17,28-32,200-207
  spanning-tree port type network
  spanning-tree guard root
  channel-group 10 mode active
  no shutdown

interface port-channel10
  description Two Way vPC to 5020s
  switchport
  switchport mode trunk
  switchport trunk allowed vlan 2-17,28-32,200-207
  spanning-tree port type network
  spanning-tree guard root
  vpc 10
</pre>

**N5K Configs**

<pre lang="plain">N5K-1
interface Ethernet1/37
  description To N7K-1->Eth1/17
  switchport mode trunk
  switchport trunk allowed vlan 2-17,28-32,200-207
  spanning-tree port type network
  channel-group 10 mode active

interface port-channel10
  switchport mode trunk
  switchport trunk allowed vlan 2-17,28-32,200-207
  spanning-tree port type network
  vpc 10

N5K-2
interface Ethernet1/37
  description To N7K-1->Eth2/17
  switchport mode trunk
  switchport trunk allowed vlan 2-17,28-32,200-207
  spanning-tree port type network
  channel-group 10 mode active

interface port-channel10
  switchport mode trunk
  switchport trunk allowed vlan 2-17,28-32,200-207
  spanning-tree port type network
  vpc 10
</pre>

Although there&#8217;s two more links to each switch, and another Nexus 7K, I&#8217;m only showing the one side and two ports, because it&#8217;s all the same from there. The configuration is replicated, to the other switches.

After some digging, and looking at the logs, we found this interesting log message;

**N7K-1 LOG**

<pre lang="plain">NK7-1 %STP-2-BRIDGE_ASSURANCE_BLOCK: Bridge Assurance blocking port port-channel10 VLAN: 2
NK7-1 %STP-2-BRIDGE_ASSURANCE_BLOCK: Bridge Assurance blocking port port-channel10 VLAN: 2
</pre>

**N5K-1 LOG**

<pre lang="plain">N5K-1 %STP-2-BRIDGE_ASSURANCE_BLOCK: Bridge Assurance blocking port Po10 VLAN: 2
</pre>

As you can see, bridge assurance is blocking the ports from forwarding, (there&#8217;s more VLANs being block, but only one is shown). After a bit, the vPC actually comes down, and shows as being connected to two different devices, which is fine for vPC, but not if it fails.

# Deeper Digging {#deeperdigging}

Futher troubleshooing, just because this is not production ready and we had time to dig for a bit. Here&#8217;s the spanning-tree counters for those port-channels.

**N7K-1 STP Counters**

<pre lang="plain">Port 4096 (port-channel10, vPC) of VLAN0002 is designated forwarding 
   &lt;Some information removed for brevity>
   ..........
   ..........
   BPDU: sent 20, received 0
</pre>

**N5K-1 STP Counters**

<pre lang="plain">Port 4096 (port-channel10, vPC) of VLAN0002 is root forwarding 
   &lt;Some information removed for brevity>
   ..........
   ..........
   BPDU: sent 0, received 20
</pre>

As you can probably see, the Nexus 7000 is sending BPDUs out the Port-channel, and the Nexus 5020 is receiving them, however, the reverse is not the same. But this doesn&#8217;t explain why both sides are having the *Bridge-assurance issue*. If we look at the other Nexus 5020, we see a problem;

**N7K-2 STP Counters**

<pre lang="plain">Port 4096 (port-channel10, vPC) of VLAN0002 is root forwarding 
   &lt;Some information removed for brevity>
   ..........
   ..........
   BPDU: sent 0, received 0
</pre>

Apparently, not all ports in the **vPC** are sending and receiving BPDUs, hence bridge-assurance detecting the loss (Thinks it&#8217;s not connected to a switch) and down goes the **vPC**.

# The Fix {#thefix}

All that really had to be done was remove bridge-assurance from the **vPC Port-channel** and all was good. While this was a good troubleshooting experience, it is documented in one of the Cisco <a href="http%3A%2F%2Fwww.cisco.com%2Fen%2FUS%2Fprod%2Fcollateral%2Fswitches%2Fps9441%2Fps9670%2FC07-572829-01_Design_N5K_N2K_vPC_DG.pdf" target="blank">Design Guides</a> for **Data Center Access with the Nexus 5000** document.

This states that **Only the vPC Primary** processes spanning-tree BPDUs, and that **Bridge-assurance** should not be enabled on **vPC** Member ports.

**Fixed Config**

<pre lang="plain">On All Switches
....
interface port-channel10
  no spanning-tree port type network
</pre>

## Summary {#summary}

I had read the above design guide, more than once, but there's some things that always escape. This is one, that probably wont escape again, and was a good troubleshooting experience. Funny enough, I had configured two-way vPCs before, and that information was probably fresh in my mind, or I just blatently forgot this time.