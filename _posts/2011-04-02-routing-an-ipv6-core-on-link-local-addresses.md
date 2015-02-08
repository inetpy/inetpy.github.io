---
title: Routing an IPv6 Core on Link-Local Addresses
author: yandy
layout: post
permalink: /2011/04/routing-an-ipv6-core-on-link-local-addresses
image: ipv6_link-local_only.png
twitter_image: ipv6_link-local_only.png
description: Is it possible to route an IPv6 core network on link-local only? Step inside and read more.
summary: Can routing an IPv6 Core on link-local addresses be done? Will IPv6 work in a network backbone that only has link-local addresses configured? For this test I’ll be using OSPFv3 but protocol itself shouldn’t matter much (on native broadcast/multicast capable interfaces) anyway. The topology I’ll be using is probably going to be the same one from now on, (or at the very least variations of the same) I kept changing it before. 
dsq_thread_id:
  - 2479760750
categories:
  - Networking
tags:
  - icmpv6
  - ipv6
  - link-local
  - ospf ipv6
  - ospfv3
  - path mtu
---
![](http://ipyandy.net/images/ipv6_banner.jpg)

Can routing an IPv6 Core on link-local addresses be done? Will IPv6 work in a network backbone that only has link-local addresses configured? For this test I’ll be using OSPFv3 but protocol itself shouldn’t matter much (on native broadcast/multicast capable interfaces) anyway. The topology I’ll be using is probably going to be the same one from now on, (or at the very least variations of the same) I kept changing it before. This is is thanks to <a title="“ioshints”" href="https://twitter.com/#!/ioshints" target="blank">Ivan Pepelnjak</a> and his awesome <a title="“webinars”" href="http://www.ioshints.info/Webinars" target="blank">webinars</a>, gave me the idea for a hack of his topology.

### Topology

Basically, a set of routers connecting to the **”core”** with various serial connections. Running OSPFv3 area 0 in the core with only IPv6 Link-Local addresses on the PPP and Frame-relay links. The remote “branches” are fully configured for IPv6 and passing information along through the backbone network. Just imagine the ABRs are set of POPs in different parts of the country, making up the backbone network, with a centralized **Core** at either the Central site or some colocation space. Purpose of this is just to verify connectivity and other possible things that may go irate in this scenario.

[<img id="img1" title="img1" src="http://ipyandy.net/assets/images/ipv6_link-local_only.png" alt="" width="" height="" />][img1]

<!--more-->

# Configuration Overview

**C1, C2, R1A, R1B** and the **core** router all have their serial interfaces and loopbacks in ***area 0***. **C1, C2, R1A** and **R1B** also have their corresponding *f0/0* interfaces in their respective areas according to the diagram. **C1** and **C2** f0/0 interfaces are in area 13, while **R1A** and **R1B** f0/0 interfaces are in area 21. Each router has a loopback interface with an IPv6 address of 2001:0:0:10::X/128 where X is outlined in the diagram. Most of the connectivity testing will be done from **C3** with the exception of a few debugs from **R1A** and the **core** router.

# The View from C1’s Perspective {#theviewfromc1sperspective}

Let’s take a look at what **C1** is seeing in this scenario to have a protocol view of what we’re working with.

#### OSPF Outputs

<pre lang="“plain">C1#sh ipv6 ospf int br
Interface    PID   Area            Intf ID    Cost  State Nbrs F/C
Lo0          1     0               14         1     LOOP  0/0
Se1/0        1     0               5          64    P2P   1/1
Fa0/0        1     13              3          1     BDR   2/2</pre>

C1#sh ipv6 ospf neigh

Neighbor ID Pri State Dead Time Interface ID Interface 10.0.0.100 0 FULL/ &#8211; 00:00:33 4 Serial1/0 10.0.0.2 1 FULL/DR 00:00:35 3 FastEthernet0/0 10.0.0.13 0 FULL/DROTHER 00:00:31 3 FastEthernet0/0

The OSPF interfaces in the backbone area are Se1/0 which connects to **core** s1/0 and the loop0 interface. Fa0/0 is in area 13 and we can also see the neighbor relationships on each of the corresponding interfaces, except for Lo0 which obviously does not have any neighbors. On S1/0 there’s 10.0.0.100 (Core) router as a neighbor, and two neighbors on Fa0/0, 10.0.0.2 being C2 and 10.0.0.13 Being C3. Now let’s take a look at the OSPFv3 routing table

#### OSPFv3 RIB

<pre lang="plain">C1#sh ipv6 route ospf

OI  2001:0:0:10::2/128 [110/1]
     via FE80::C805:FFF:FEEE:8, FastEthernet0/0
O   2001:0:0:10::3/128 [110/128]
     via FE80::C80A:FFF:FEFC:0, Serial1/0
O   2001:0:0:10::4/128 [110/128]
     via FE80::C80A:FFF:FEFC:0, Serial1/0
O   2001:0:0:10::13/128 [110/1]
     via FE80::C807:FFF:FEEE:8, FastEthernet0/0
OI  2001:0:0:10::21/128 [110/129]
     via FE80::C80A:FFF:FEFC:0, Serial1/0
OI  2001:10:1:21::/64 [110/129]
     via FE80::C80A:FFF:FEFC:0, Serial1/0
OI  2001:10:1:22::/64 [110/129]
     via FE80::C80A:FFF:FEFC:0, Serial1/0</pre>

From here, we can see routes to R1A and R1B’s respective loopbacks, :3 and :4. Also, we see routes to the segment connecting R1A, R1B and RB1 together, being 2001:10:1:21::/64. One thing to note, even for segments that have full IPv6 configuration (globally routable addresses) such as the segment directly connected to C2 and C3. The next hop addresses for their respective loopbacks is still a Link-Local address, not the interface Global IPv6 address.

### C3’s View of the Network

Moving on where most of our connectivity testing will happen, which is from **C3**. Let’s take a look at it’s OSPFv3 relationships to interfaces and neighbors, just to make sure we have everything set.

#### C3’s OSPF Information and IPV6 interfaces

<pre lang="plain">C3#sh ipv6 ospf int br
Interface    PID   Area            Intf ID    Cost  State Nbrs F/C
Lo0          1     13              14         1     LOOP  0/0
Fa0/0        1     13              3          1     DROTH 2/2

C3#sh ipv6 ospf neigh

Neighbor ID     Pri   State           Dead Time   Interface ID    Interface
10.0.0.1          1   FULL/BDR        00:00:36    3               FastEthernet0/0
10.0.0.2          1   FULL/DR         00:00:34    3               FastEthernet0/0

C3#sh ipv6 int br    
FastEthernet0/0            [up/up]
    FE80::C807:FFF:FEEE:8
    2001:10:1:13::13
Loopback0                  [up/up]
    FE80::C807:FFF:FEEE:8
    2001:0:0:10::13</pre>

So there’s Fa0/0 and Lo0 which are both in area 13 (all interfaces in area 13). Then there’s two neighbors, both on Fa0/0, **C1** and **C2**. One being DR and the other the BDR for the segment. Just like OSPFv2 (for IPv4) I’ve set the priority on interface Fa0/0 to 0 (zero) to make sure that **C3** never becomes the DR for that network.

#### C3’s OSPFv3 RIB

<pre lang="plain">C3#sh ipv6 route ospf

OI  2001:0:0:10::1/128 [110/1]
     via FE80::C806:FFF:FEEE:8, FastEthernet0/0
OI  2001:0:0:10::2/128 [110/1]
     via FE80::C805:FFF:FEEE:8, FastEthernet0/0
OI  2001:0:0:10::3/128 [110/129]
     via FE80::C805:FFF:FEEE:8, FastEthernet0/0
     via FE80::C806:FFF:FEEE:8, FastEthernet0/0
OI  2001:0:0:10::4/128 [110/129]
     via FE80::C805:FFF:FEEE:8, FastEthernet0/0
     via FE80::C806:FFF:FEEE:8, FastEthernet0/0
OI  2001:0:0:10::21/128 [110/130]
     via FE80::C806:FFF:FEEE:8, FastEthernet0/0
     via FE80::C805:FFF:FEEE:8, FastEthernet0/0
OI  2001:10:1:21::/64 [110/130]
     via FE80::C806:FFF:FEEE:8, FastEthernet0/0
     via FE80::C805:FFF:FEEE:8, FastEthernet0/0</pre>

Much of the same information exists in **C3’s** RIB, with the addition of both **C1’s** and **C2’s** loop0 addresses in the table.

Just to make sure that we have all the information we need, let’s take a look at **RB1’s** routing table. So that we can see all routes are propagated throughout the network and that it can respond to the tests from **C3**.

### RB1’s Routing Table

<pre lang="plain">RB1#sh ipv6 route ospf

OI  2001:0:0:10::1/128 [110/129]
     via FE80::C804:FFF:FEEE:8, FastEthernet0/0
     via FE80::C803:FFF:FEEE:8, FastEthernet0/0
OI  2001:0:0:10::2/128 [110/129]
     via FE80::C804:FFF:FEEE:8, FastEthernet0/0
     via FE80::C803:FFF:FEEE:8, FastEthernet0/0
OI  2001:0:0:10::3/128 [110/1]
     via FE80::C803:FFF:FEEE:8, FastEthernet0/0
OI  2001:0:0:10::4/128 [110/1]
     via FE80::C804:FFF:FEEE:8, FastEthernet0/0
OI  2001:0:0:10::13/128 [110/130]
     via FE80::C804:FFF:FEEE:8, FastEthernet0/0
     via FE80::C803:FFF:FEEE:8, FastEthernet0/0
OI  2001:10:1:13::/64 [110/130]
     via FE80::C804:FFF:FEEE:8, FastEthernet0/0
     via FE80::C803:FFF:FEEE:8, FastEthernet0/0</pre>

We do see all of Area 13’s information the 2001:10:1:13::/64 and the route to **C3’s** looback 2001:0:0:10::3/128. We should be ready now to begin our testing and experimenting. To recap again, the backbone facing network only has link-local addresses enabled with the command “ipv6 enable” and here’s the output to prove it.

### The Core Router’s Interfaces

<pre lang="plain">Core#sh ipv6 int br   
FastEthernet0/0            [administratively down/down]
    unassigned
Serial1/0                  [up/up]
    FE80::C80A:FFF:FEFC:0
Serial1/1                  [up/up]
    FE80::C80A:FFF:FEFC:0
Serial1/7.100              [up/up]
    FE80::C80A:FFF:FEFC:0
Serial1/7.101              [up/up]
    FE80::C80A:FFF:FEFC:0
Loopback0                  [up/up]
    FE80::C80A:FFF:FEFC:0</pre>

### C1’s IPv6 Enabled Interfaces

<pre lang="plain">C1#sh ipv6 int br
FastEthernet0/0            [up/up]
    FE80::C806:FFF:FEEE:8
    2001:10:1:13::1
Serial1/0                  [up/up]
    FE80::C806:FFF:FEEE:8    &lt;&lt;&lt;&lt; — Core facing interface
Loopback0                  [up/up]
    FE80::C806:FFF:FEEE:8
    2001:0:0:10::1</pre>

### C2’s IPv6 Enabled Interfaces

<pre lang="plain">C2#sh ipv6 int br
FastEthernet0/0            [up/up]
    FE80::C805:FFF:FEEE:8
    2001:10:1:13::2
Serial1/0                  [up/up]    &lt;&lt;&lt;&lt; — Core facing interface
    FE80::C805:FFF:FEEE:8
Loopback0                  [up/up]
    FE80::C805:FFF:FEEE:8
    2001:0:0:10::2</pre>

### R1A’s IPv6 Enabled Interfaces

<pre lang="plain">R1A#sh ipv6 int br
FastEthernet0/0            [up/up]
    FE80::C803:FFF:FEEE:8
    2001:10:1:21::1
Serial1/0.100              [up/up]
    FE80::C803:FFF:FEEE:8    &lt;&lt;&lt;&lt; — Core facing interface
Loopback0                  [up/up]
    FE80::C803:FFF:FEEE:8
    2001:0:0:10::3</pre>

### R1B’s IPv6 Enabled Interfaces

<pre lang="plain">R1B#sh ipv6 int br
FastEthernet0/0            [up/up]
    FE80::C804:FFF:FEEE:8
    2001:10:1:21::2
Serial1/0.101              [up/up]
    FE80::C804:FFF:FEEE:8    &lt;&lt;&lt;&lt; — Core facing interface
Loopback0                  [up/up]
    FE80::C804:FFF:FEEE:8
    2001:0:0:10::4</pre>

### Testing Begins

So now we can begin our testing from **C3**, let’s begin with a simple ping to **RB1’s** looback address of (2001:0:0:10::21).

#### First Ping Test

<pre lang="plain">C3#ping 2001:0:0:10::21 source loop0 

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:0:0:10::21, timeout is 2 seconds:
Packet sent with a source address of 2001:0:0:10::13
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/7/12 ms</pre>

So from the looks of it, we have reachability end-to-end. That should be it right, we’ve proven and everything works? If only things were that easy eh? But, they never are, so let’s try a traceroute from to that same address and see what we get.

#### First Trace Route

<pre lang="plain">C3#trace ipv6                        

Target IPv6 address: 2001:0:0:10::21
Source address: 2001:0:0:10::13
Insert source routing header? [no]: 
Numeric display? [no]: 
Timeout in seconds [3]: 
Probe count [3]: 
Minimum Time to Live [1]: 
Maximum Time to Live [30]: 
Priority [0]: 
Port Number [0]: 
Type escape sequence to abort.
Tracing the route to 2001:0:0:10::21

  1 2001:10:1:13::2 8 msec
    2001:10:1:13::1 0 msec
    2001:10:1:13::2 0 msec
  2  *  *  * 
  3 2001:0:0:10::3 24 msec 12 msec 12 msec
  4 2001:0:0:10::21 12 msec 4 msec 12 msec</pre>

Wow, so we have end to end connection; but what happened there in the middle, our second hop? Looking at the diagram this should be the **core** router. Maybe there’s no route back to back to **C3’s** loopback interface on the **core** router? Let’s take a look:

#### Core Router’s RIB

<pre lang="plain" line="“1”">Core#sh ipv6 route ospf</pre>

O 2001:0:0:10::1/128 [110/64] via FE80::C806:FFF:FEEE:8, Serial1/0 O 2001:0:0:10::2/128 [110/64] via FE80::C805:FFF:FEEE:8, Serial1/1 O 2001:0:0:10::3/128 [110/64] via FE80::C803:FFF:FEEE:8, Serial1/7.100 O 2001:0:0:10::4/128 [110/64] via FE80::C804:FFF:FEEE:8, Serial1/7.101 OI 2001:0:0:10::13/128 [110/65] via FE80::C805:FFF:FEEE:8, Serial1/1 via FE80::C806:FFF:FEEE:8, Serial1/0 OI 2001:0:0:10::21/128 [110/65] via FE80::C803:FFF:FEEE:8, Serial1/7.100 via FE80::C804:FFF:FEEE:8, Serial1/7.101 OI 2001:10:1:13::/64 [110/65] via FE80::C805:FFF:FEEE:8, Serial1/1 via FE80::C806:FFF:FEEE:8, Serial1/0 OI 2001:10:1:21::/64 [110/65] via FE80::C803:FFF:FEEE:8, Serial1/7.100 via FE80::C804:FFF:FEEE:8, Serial1/7.101

If you look closely you see that, well, there is a route back to **C3’s** loopback interface. And if there weren’t, we wouln’t get return traffic to begin with. So, what’s the issue then? And why is it even important if we have end to end connectivity? There are no packet filters, or policy routing in place.

So, let’s take a look a little deeper and look at some debugging output from the **core** router.

#### Core Debugging

<pre lang="plain">*Apr  1 22:10:15.169: IPv6-Fwd: Destination lookup for 2001:0:0:10::21 : i/f=Serial1/7.101, nexthop=FE80::C804:FFF:FEEE:8
*Apr  1 22:10:15.173: IPV6: source 2001:0:0:10::13 (Serial1/0)
*Apr  1 22:10:15.173:       dest 2001:0:0:10::21 (Serial1/7.101)
*Apr  1 22:10:15.173:       traffic class 0, flow 0x0, len 48+4, prot 17, hops 1, bad hop count
*Apr  1 22:10:15.173: IPv6-Sas: SAS picked source FE80::C80A:FFF:FEFC:0 for 2001:0:0:10::13
*Apr  1 22:10:15.173: IPv6-Fwd: nexthop FE80::C805:FFF:FEEE:8,
Core#
*Apr  1 22:10:18.189: IPv6-Fwd: Destination lookup for 2001:0:0:10::21 : i/f=Serial1/7.100, nexthop=FE80::C803:FFF:FEEE:8
*Apr  1 22:10:18.193: IPV6: source 2001:0:0:10::13 (Serial1/0)
*Apr  1 22:10:18.197:       dest 2001:0:0:10::21 (Serial1/7.100)
*Apr  1 22:10:18.197:       traffic class 0, flow 0x0, len 48+4, prot 17, hops 1, bad hop count
*Apr  1 22:10:18.197: IPv6-Sas: SAS picked source FE80::C80A:FFF:FEFC:0 for 2001:0:0:10::13
*Apr  1 22:10:18.197: IPv6-Fwd: nexthop FE80::C805:FFF:FEEE:8,
Core#
*Apr  1 22:10:21.161: IPv6-Fwd: Destination lookup for 2001:0:0:10::21 : i/f=Serial1/7.101, nexthop=FE80::C804:FFF:FEEE:8
*Apr  1 22:10:21.165: IPV6: source 2001:0:0:10::13 (Serial1/0)
*Apr  1 22:10:21.165:       dest 2001:0:0:10::21 (Serial1/7.101)
*Apr  1 22:10:21.169:       traffic class 0, flow 0x0, len 48+4, prot 17, hops 1, bad hop count
*Apr  1 22:10:21.173: IPv6-Sas: SAS picked source FE80::C80A:FFF:FEFC:0 for 2001:0:0:10::13
*Apr  1 22:10:21.177: IPv6-Fwd: nexthop FE80::C805:FFF:FEEE:8,</pre>

Taking a look at the end of the fourth line from each block of output, there’s this *bad hop count* output. What does that mean? And why are we seeing that? You would think that’s the issue, however, this is just how trace-route works, the hop-count is incrementally set to make sure that every router along the path answers, since the destination is beyond the hop-count specified in the packet and delivers a message back to the source.

But, let’s dig deeper, and take a look at what **C1** is seeing from the **core** as it receives those packets returning from the *core* router.

#### C1’s Debug Output

<pre lang="plain">*Apr  1 22:29:06.885: IPv6-Fwd: Destination lookup for 2001:0:0:10::13 : i/f=FastEthernet0/0, nexthop=FE80::C807:FFF:FEEE:8
*Apr  1 22:29:06.885: IPV6: source FE80::C80A:FFF:FEFC:0 (Serial1/0)
*Apr  1 22:29:06.889:       dest 2001:0:0:10::13 (FastEthernet0/0)
*Apr  1 22:29:06.889:       traffic class 0, flow 0x0, len 96+4, prot 58, hops 63, forwarding
*Apr  1 22:29:06.889: IPv6-Fwd: Beyond scope of source addr
C1#ess
C1#
*Apr  1 22:29:09.889: IPv6-Fwd: Destination lookup for 2001:0:0:10::13 : i/f=FastEthernet0/0, nexthop=FE80::C807:FFF:FEEE:8
*Apr  1 22:29:09.889: IPV6: source FE80::C80A:FFF:FEFC:0 (Serial1/0)
*Apr  1 22:29:09.893:       dest 2001:0:0:10::13 (FastEthernet0/0)
*Apr  1 22:29:09.893:       traffic class 0, flow 0x0, len 96+4, prot 58, hops 63, forwarding
*Apr  1 22:29:09.893: IPv6-Fwd: Beyond scope of source address
C1#
*Apr  1 22:29:12.913: IPv6-Fwd: Destination lookup for 2001:0:0:10::13 : i/f=FastEthernet0/0, nexthop=FE80::C807:FFF:FEEE:8
*Apr  1 22:29:12.913: IPV6: source FE80::C80A:FFF:FEFC:0 (Serial1/0)
*Apr  1 22:29:12.913:       dest 2001:0:0:10::13 (FastEthernet0/0)
*Apr  1 22:29:12.913:       traffic class 0, flow 0x0, len 96+4, prot 58, hops 63, forwarding
*Apr  1 22:29:12.913: IPv6-Fwd: Beyond scope of source address</pre>

Let’s dissect the output for a second, we see the router looking up a next-hop for *2001:0:0:10::13* which is **C3**. Then finding the next-hop of FE80::C807:FFF:FEEE:8 which looking back at an earlier output, we know that this is **C3’s** link-local address attached to Fa0/0. The the source of the packet, which is *FE80::C80A:FFF:FFFC:0*, this being the *core* router’s link-local attached to Se1/0. Then, we see a few parameters dealing with protocol information and a “forwarding” keyword. But, right after that it’s an ICMPv6 message ***IPv6-Fwd: Beyond scope of source addr***, according to the RFC this means the destination is beyond the scope of the source, which of course is a link-local source with a scope of 1. It cannot be forwarded or used beyond that link, hence the name *link-local*.

#### From <a href="http://www.ietf.org/rfc/rfc4443.txt" target="blank">RFC4443</a> 

### ICMPv6

> A Destination Unreachable message SHOULD be generated by a router, or  
> by the IPv6 layer in the originating node, in response to a packet  
> that cannot be delivered to its destination address for reasons other  
> than congestion. (An ICMPv6 message MUST NOT be generated if a  
> packet is dropped due to congestion.)
> 
> If the reason for the failure to deliver is that the destination is  
> beyond the scope of the source address, the Code field is set to 2.  
> This condition can occur only when the scope of the source address is  
> smaller than the scope of the destination address (e.g., when a  
> packet has a link-local source address and a global-scope destination  
> address) and the packet cannot be delivered to the destination  
> without leaving the scope of the source address.

We know that the **core** router has no IPv6 Global addresses, since this was the point of the exercise, to see if it’s possible to route IPv6 though a Link-local only backbone. And so far technically it can, just traceroutes break, or anything directed at the routers in the backbone themselves. Which in essence is not that big of a deal, since most things traverse the backbone and are not directed at it. Or is it? There’s a big thing here which I’ll explain later.

I’m going to add an IPv6 address of *2001:0:0:10::100/128* to the **core** router’s loopback0 interface and try that traceroute again.

### Core Configuration

<pre lang="plain">Internet#sh run int loop0
Building configuration...

Current configuration : 137 bytes
!
interface Loopback0
 ip address 10.0.0.100 255.255.255.255
 ipv6 address 2001:0:0:10::100/128
 ipv6 enable
 ipv6 ospf 1 area 0
 !
end</pre>

### More Testing From C3

Let’s do those traces again, since the **core** now has a valid routable IPv6 address. While not connected to any serial link, it’s only on it’s loopback interface.

<pre lang="plain">C3#trace ipv6

Target IPv6 address: 2001:0:0:10::21
Source address: 2001:0:0:10::13
Insert source routing header? [no]: 
Numeric display? [no]: 
Timeout in seconds [3]: 
Probe count [3]: 
Minimum Time to Live [1]: 
Maximum Time to Live [30]: 
Priority [0]: 
Port Number [0]: 
Type escape sequence to abort.
Tracing the route to 2001:0:0:10::21

  1 2001:10:1:13::1 32 msec 8 msec 4 msec
  2 2001:0:0:10::100 8 msec 8 msec 8 msec
  3 2001:0:0:10::3 8 msec 12 msec 8 msec
  4 2001:0:0:10::21 4 msec 16 msec 8 msec</pre>

Looking at this output, now the second hop is visible with a response from the **core’s** loopback address. Let’s see the debugs on the core to see how this happened.

### More Core Debugging

<pre lang="plain">*Apr  1 23:53:27.287: IPv6-Fwd: Destination lookup for 2001:0:0:10::21 : i/f=Serial1/7.100, nexthop=2001:0:0:10::21
*Apr  1 23:53:27.291: IPV6: source 2001:0:0:10::13 (Serial1/0)
*Apr  1 23:53:27.291:       dest 2001:0:0:10::21 (Serial1/7.100)
*Apr  1 23:53:27.295:       traffic class 0, flow 0x0, len 48+4, prot 17, hops 1, bad hop count
*Apr  1 23:53:27.299: IPv6-Sas: SAS picked source 2001:0:0:10::100 for 2001:0:0:10::13
*Apr  1 23:53:27.303: IPv6-Fwd: Destination lookup for 2001:0:0:10::13 : i/f=Serial1/0, nexthop=2001:0:0:10::13
*Apr  1 23:53:27.307: IPV6: source 2001:0:0:10::100 (local)
*Apr  1 23:53:27.307:       dest 2001:0:0:10::13 (Serial1/0)
*Apr  1 23:53:27.311:       traffic class 0, flow 0x0, len 96+0, prot 58, hops 64, originating
*Apr  1 23:53:27.311: IPv6-Fwd: Sending on Serial1/0
!
*Apr  1 23:53:27.323: IPv6-Fwd: Destination lookup for 2001:0:0:10::21 : i/f=Serial1/7.100, nexthop=2001:0:0:10::21
*Apr  1 23:53:27.327: IPV6: source 2001:0:0:10::13 (Serial1/0)
*Apr  1 23:53:27.327:       dest 2001:0:0:10::21 (Serial1/7.100)
*Apr  1 23:53:27.327:       traffic class 0, flow 0x0, len 48+4, prot 17, hops 1, bad hop count
*Apr  1 23:53:27.327: IPv6-Sas: SAS picked source 2001:0:0:10::100 for 2001:0:0:10::13
*Apr  1 23:53:27.327: IPv6-Fwd: Destination lookup for 2001:0:0:10::13 : i/f=Serial1/0, nexthop=2001:0:0:10::13
*Apr  1 23:53:27.327: IPV6: source 2001:0:0:10::100 (local)
*Apr  1 23:53:27.327:       dest 2001:0:0:10::13 (Serial1/0)
*Apr  1 23:53:27.327:       traffic class 0, flow 0x0, len 96+0, prot 58, hops 64, originating
*Apr  1 23:53:27.327: IPv6-Fwd: Sending on Serial1/0
!
*Apr  1 23:53:27.335: IPv6-Fwd: Destination lookup for 2001:0:0:10::21 : i/f=Serial1/7.100, nexthop=2001:0:0:10::21
*Apr  1 23:53:27.335: IPV6: source 2001:0:0:10::13 (Serial1/0)
*Apr  1 23:53:27.335:       dest 2001:0:0:10::21 (Serial1/7.100)
*Apr  1 23:53:27.335:       traffic class 0, flow 0x0, len 48+4, prot 17, hops 1, bad hop count
*Apr  1 23:53:27.335: IPv6-Sas: SAS picked source 2001:0:0:10::100 for 2001:0:0:10::13
*Apr  1 23:53:27.335: IPv6-Fwd: Destination lookup for 2001:0:0:10::13 : i/f=Serial1/0, nexthop=2001:0:0:10::13
*Apr  1 23:53:27.335: IPV6: source 2001:0:0:10::100 (local)
*Apr  1 23:53:27.335:       dest 2001:0:0:10::13 (Serial1/0)
*Apr  1 23:53:27.335:       traffic class 0, flow 0x0, len 96+0, prot 58, hops 64, originating
*Apr  1 23:53:27.335: IPv6-Fwd: Sending on Serial1/0</pre>

Now we can clearly see that the fourth line from each block, is sourcing from it’s loopback0 interface. Apparently, all we need is **one** valid IPv6 address that’s routable and the router will source from there. Now, obviously is a good idea this address is also reachable, which means including it in your IGP.

But why’s it important for ICMPv6 to be available for things like traces and pings. Well, it’s not for that exactly, but one very important part called, Path MTU Discovery, and according to the RFC this is a must in IPv6.

From <a title="“ICMPv6”" href="http://www.ietf.org/rfc/rfc4443.txt" target="blank">RFC4443</a> ICMPv6

> A Packet Too Big MUST be sent by a router in response to a packet  
> that it cannot forward because the packet is larger than the MTU of  
> the outgoing link. The information in this message is used as part  
> of the Path MTU Discovery process [PMTU].
> 
> Originating a Packet Too Big Message makes an exception to one of the  
> rules as to when to originate an ICMPv6 error message. Unlike other  
> messages, it is sent in response to a packet received with an IPv6  
> multicast destination address, or with a link-layer multicast or  
> link-layer broadcast address.

The first sentence, of the first paragraph states in “CAPS” that a router MUST send a packet too big message if it cannot forward the packet do to and MTU size too big. In IPv4 this was an optional thing, and normally built into applications themselves. With IPv6 this is something built right on the protocol itself and it’s a “MUST”. You can see where applications can get broken if Path MTU cannot be resolved.

Although the second paragraph states, the one of the rules about link-local (non-routable) addresses is broken. This specifies only IPv6 multicast link-local address and not unicast sources.

From <a href="http://www.rfc-editor.org/rfc/rfc2460.txt" target="blank">RF2460</a> IPv6 Specification 

> The Fragment header is used by an IPv6 source to send a packet larger  
> than would fit in the path MTU to its destination. (Note: unlike  
> IPv4, fragmentation in IPv6 is performed only by source nodes, not by  
> routers along a packet’s delivery path

This clearly states that routers in the destination path cannot fragment the packet an must be done at the source. Hence the need for the source to be able to receive all ICMPv6 messages sent, and this would’ve been broken if we left the network how it previously was.

### In Summary

While IPv6 can be routed end-to-end for proper source addresses through a backbone consisting of Link-Local addresses only. This would essentially only work in a network that never has MTU too big issues, and it’s a bad idea, which we saw why. Now adding one single globally routable address to the core routers can fix this issue. Adding it to a loopback address and making it reachable can guarantee delivery. But, is this a good design to begin with? Maybe, maybe not. Tunneling IPv6 over IPv4 would probably be better as the backbone becomes IPv6 ready.

  * It can be useful if a couple of sites need to be brought up on IPv6, and there’s no time for proper addressing plan on the backbone
  * Let’s face it, not all can afford a multi-router lab scenario to simulate production, and this is not a perfect world
  * In most companies, things need to be done “NOW” when the “right” person asks
  * Rather than have a crappy IPv6 design, this can be used to plug holes while the design is brought up, adding addresses to interfaces along the path “should” not break anything.
  * If you need to make sure your backbone would work with IPv6, and again no $$$ for lab or proof of concept (in theory) a Unique Local IPv6 address should work as well, since it would be routable within the organization. And, there’s no need to have your own block of IPv6 addresses for testing.
  * It’s easier to remove one IPv6 from a loopback address, than form hundreds or thousands of connected backbone links.

While it should probably be avoided, at least we know it works with at least ONE address being routable. There were no applications or IPv6 addresses harmed in the making of this post.

 [img1]: http://ipyandy.net/assets/images/ipv6_link-local_only.png