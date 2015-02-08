---
title: vPC and VSS features and Comparison
author: yandy
layout: post
permalink: /2011/01/vpc-and-vss-features-and-comparison/
image: generic_banner.jpg
twitter_image: generic_banner.jpg
description: 
summary: Seems like other than IPv6 allot of the talk lately (in the Datacenter anyways) is about MEC, or multi-chassis etherchannel. Using something like this in the aggregation part of the Datacenter not just limited to physically stackable access switches, such as the Catalyst 3750s.
categories:
  - Networking
tags:
  - 6500
  - catalyst
  - Nexus
  - switching
  - vpc
  - vss
---
<hr>
![](http://ipyandy.net/images/generic_banner.jpg)
<hr>

Seems like other than IPv6 allot of the talk lately (in the Datacenter anyways) is about MEC, or multi-chassis etherchannel. Using something like this in the aggregation part of the Datacenter not just limited to physically stackable access switches, such as the Catalyst 3750s. There's been plenty of posts about this subject, some of the best at Ivan's blog over at <a href="http://blog.ioshints.inf" target="_blank">ioshints.info</a>, but I thought I'll add my own thoughts and what I've come to learn.

Bit of a disclaimer, this isn't exactly a primer, so some terms may be used and not defined.. allot of information on getting started on these technologies is available at the Cisco site.

**VSS (Virtual Switching System) and vPC (Virtual Port-Channel)**

To begin with, VSS is a Catalyst 6500 implementation of the so called MEC (multi-chassis etherchannel) or MLAG (multi-chassis link aggregation). And vPC is a Nexus switch implementation of the same. While they both strive to pretty much do the same job, the way they do it is all but completely different.

Comparing both below, more details to come. 

[<img id="img1" title="img1" src="http://ipyandy.net/assets/images/vPC_VSS.png" alt="" width="" height="" />][img1]

<!--more-->

The picture above depicts the two precesses, and I'll try to explain more in the next few sections.

**Control Plane - VSS**

VSS can have a maximum of two switches and any VSS configuration. Uses a type of unity of the control plane, in other words the two switches are managed by the ONE active switch and all configuration and state is synchronized to the standby switch over the VSL (virtual switch link). All routing, spanning-tree, snmp processing, basically anything punted to the CPU is handled by the Active switch.

Both switches become this ONE virtual switch and a the brain is shared between both. It becomes basically like any other stack, although the background of how it's done is different.

**Control Plane &#8211;  vPC**

In a vPC environment the control plane is kept separate, meaning the Nexus switches are still managed and have individual processing of control information (spanning-tree, ospf adjacencies, eigrp.. and so on). State is exchanged through the vpc peer-link, which is the equivalent of the VSS virtual switch link, but acts as a normal Layer-2 trunk (preferred).

**Data Plane VSS and vPC**

The Data planes for both of these is somewhat the same, individual and separate. Each switch handles the forwarding of data individually of the other switch. So although in VSS control traffic is handled by the one active switch, once the forwarding tables are built, the data is forwarded locally and does not need to traverse the VSL.

The same with vPC, all control information and data is separate of each other. Once state is exchanged all forwarding is done independently of each other, unless a single home situation is encountered and traffic needs to be forwarded to the vPC peer-link in order to head to it's final destination.

**VSS Hardware Requirements**

In order for the two switches to participate in a VSS, they need to synchronize hardware capabilities. The only Supervisor to support VSS is the VS-S720-10G with PFC3. Also the line cards need to be WS-X67xx models, DFC3-C or DFC3-CXLs are not required but recommended. Also as of IOS 12.2.SXI(33) support was added for the FWSM and ACE hardware line cards.

**vPC Hardware Requirements**

Unlike VSS, the vPC requirements are not so stringent, for one exception. It does require the peer-link to be at least one 10G port, nothing slower, in other words, the peer-link will not form on 1G. Now, vPC enabled ports can be any 1G or 10G ports to create an MEC to other devices.

**Detecting Dual-Active Scenario in VSS**

If a loss of the VSL occurs, then VSS applies a couple of procedures to find if it's just a link failure or a complete neighboring device failure. First of, you can run EPaGP or Enhanced-PaGP over an MEC link, sending some encapsulation including it's switch ID, they can detect if the other side is still active. However, this EPaGP feature is required on the downstream device as well. If the active switch in the VSS does not see it's neighboring ID in one of the encapsulated PAGP messages, then it removes itself from all VSS duties.

The second is running BFD (Bidirectional Forwarding Direction) on up to four interfaces. Now, a whole summary on BFD can take a book pretty much by itself, but it's quick keep-alive messages are probably my preferred way of keeping dual-active scenarios at bay.

**EDIT:**

As one of the comments by &#8220;Cliff&#8221; pointed out, there's also a way of detecting dual-active scenario, this is done by a dedicated link running fast hellos.  
Details here: <a rel="nofollow" href="http://www.cisco.com/en/US/docs/switches/lan/catalyst6500/ios/12.2SX/configuration/guide/vss.html#wp1115311">http://www.cisco.com/en/US/docs/switches/lan/catalyst6500/ios/12.2SX/configuration/</a>

**HSRP and VSS**

Since VSS acts as ONE logical switch, HSRP or any kind of FHRP (Fist-hop Redundancy Protocol) is not necessary. Configuration of an IP address to an MEC is like applying it to one switch. And if trying to configure a port on both switches, with the same subnet and configuring HSRP outside of an MEC, well it's not permitted, as the subnet will overlap interfaces.. remember (ONE LOGICAL SWITCH).

**HSRP and vPC**

HSRP is configured exactly the way you would on any other platform (with the differences in command structure). It has been enhanced using the vPC hashing algorithms so that both the active and standby HSRP routers can forward a packet and load balance. The ARP replies to the virtual mac-address are still done at the active HSRP router.

**Spanning-tree in vPC and VSS**

Although spanning-tree is still a recommended configuration on either case, it's capabilities are not leveraged. They're only in place incase of a failure to limit the failure domain. Since there's no loop in the network and all links forward at the same time, spanning-tree has no bearing.

**Final thoughts**

MECs on a VSS system can be either Layer-2 or Layer-3, while on vPC only Layer-2 ether-channels are supported as MECs. It's probably the one advantage that VSS may have over vPC, the unrestricted use of Layer-2 or Layer-3. I personally much like the separate control plane mechanism on the Nexus and it's vPC. The fact of having an active supervisor seating there doing nothing but waiting for a failure to take over on a fully loaded box, gives me the creeps.

There are some scenarios where one of the Sups on the Standby switch acts as a DFC card. This requires Quad processing or 4 Sups in total, two on each switch. Which if you ask me seems like a waste of resources and with dual switches at the aggregation level, 4 Sups may be overkill.

Some of Cisco's push for VSS is that it simplifies the architecture by providing less management points. This is done through the virtualization of the Cat6500 switches. However, in some cases this may add more complexity in configuration and know-how to really be a benefit. It can also add complexity to troubleshooting unknown scenarios. It all comes down to requirements, and both vPC and VSS have their use, but not always a necessity even if appealing to the masses.

[img1]: http://ipyandy.net/assets/images/vPC_VSS.png