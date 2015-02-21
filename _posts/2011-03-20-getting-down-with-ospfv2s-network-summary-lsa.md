---
title: 'The OSPFv2 Network Summary LSA - Type-3'
author: yandy
layout: post
permalink: /2011/03/getting-down-with-ospfv2s-network-summary-lsa
image: ospf_banner_generic.jpg
twitter_image: ospf_banner_generic.jpg
description: We discuss the OSPFv2 Network Summary LSA (type-3) in detail.
summary: Well, what is the Summary LSA? It's an LSA flooded throughout the backbone area, which describes networks in other areas. Originated only by ABRs (Area Border Routers) and not flooded beyond the scope of the ABR's areas. What makes this LSA unique is, well, the SPF algorithm is not performed on it, the ABR simply adds the cost of the LSA + the cost to get to the neighboring ABR.
dsq_thread_id:
  - 2479813040
categories:
  - Networking
tags:
  - ccie
  - lsa-type-3
  - network-summary-lsa
  - ospf-database
  - summary-lsa
---
--------------------------------------------------------------------------
![](http://inetpy.com/images/ospf_banner_generic.jpg)
--------------------------------------------------------------------------

Well, what is the Summary LSA? It's an LSA flooded throughout the backbone area, which describes networks in other areas. Originated only by ABRs (Area Border Routers) and not flooded beyond the scope of the ABR's areas. What makes this LSA unique is, well, the SPF algorithm is not performed on it, the ABR simply adds the cost of the LSA + the cost to get to the neighboring ABR. This is a bit longer of a post than most my previous one’s, but hopefully it’s also useful for some.

Take this very simplistic example below:  
[<img class="alignnone" alt="" src="http://inetpy.com/assets/images/7946942c53ea1113b14c25472599014a.png" />][1]

R1 is advertising it's directly connected link of &#8220;192.168.100.0/24&#8243; to ABR1. Assuming the directly connected cost of the link that R1 is advertising has a cost of one, and the link between R1 and ABR1 is also one. Upon receiving this update, ABR1 has a route to 192.168.100.0/24 with a cost of two.

<!--more-->

ABR1 then prepares and sends a LSA type-3 advertisement for 192.168.100.0/24 to ABR2 with a cost of two. Since the metric from ABR1 to ABR2 is one as well, ABR2 receives the LSA3 update, and adds it's cost to get to ABR1, adding up to a total cost of three. And passing this update up to R2 as another LSA3 originated from ABR2 with a cost of three, R2 then just adds it's cost to ABR2 adding up to a total combined cost of four.

In this example all links have a cost of one for simplicity. This is why OSPF is said to have distance-vector type behavior between areas. Since, no trees are calculated and simply costs are added on top of each other to calculate metrics from one area to another. Only single areas have full views of their entire topology.

#### Getting Dirty With Router Outputs

Now, let's examine this in a proper configuration scenario. All links have been manually configured to have a cost of one again, to keep things simple. But viewing outputs of what's going on, will give a better feel for the inner workings.

Here's the topology we'll be working with:  
[<img class="alignnone" alt="" src="http://inetpy.com/assets/images/24e635c09a840306a4625f7bb4eb28fb.png" width="603" height="382" />][2]

BB1 is advertising it’s loopback0 interface 101.101.101.0/24 to area 100 with a cost of one. The reason a /24 is being advertised and not the default /32 for loopbacks, is that I configured “ip ospf network point-to-point” forcing the real mask to be advertised. When we look at the output for **R1**, we can see the Router LSA type-1 advertised by BB1 (101.101.101.254) including it’s loopback network.

<pre lang="plain">R1(config)#do sh ip ospf data router 101.101.101.254

            OSPF Router with ID (200.0.0.1) (Process ID 1)

                Router Link States (Area 100)

  LS age: 521
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 101.101.101.254
  Advertising Router: 101.101.101.254
  LS Seq Number: 8000002A
  Checksum: 0xF4B5
  Length: 48
  Number of Links: 2

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 101.101.101.0
     (Link Data) Network Mask: 255.255.255.0
      Number of TOS metrics: 0
       TOS 0 Metrics: 1</pre>

We can also see the same output from **R2**:

<pre lang="plain">R2(config)#do sh ip ospf data router 101.101.101.254

            OSPF Router with ID (200.0.0.2) (Process ID 1)

                Router Link States (Area 100)

  LS age: 695
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 101.101.101.254
  Advertising Router: 101.101.101.254
  LS Seq Number: 80000041
  Checksum: 0xC6CC
  Length: 48
  Number of Links: 2

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 101.101.101.0
     (Link Data) Network Mask: 255.255.255.0
      Number of TOS metrics: 0
       TOS 0 Metrics: 1</pre>

On **R2** we notice the same information, link-state from 101.101.101.254 advertised with a metric of 1. Router LSA’s are area local in scope, which means they get flooded throughout an area. If we look at two more LSA Type-1’s, one from **R1** and another originated from **R2** itself then at **R2’s** Summary Link States (LSA Type-3). We can figure out how it comes up with a metric of three for the 101.101.101.0/24 network.

Output from **R2** looking at **R1’s** LSA1 advertisement.

<pre lang="plain">R2(config)#do sh ip ospf data router 200.0.0.1

            OSPF Router with ID (200.0.0.2) (Process ID 1)

                Router Link States (Area 100)

  LS age: 1258
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 200.0.0.1
  Advertising Router: 200.0.0.1
  LS Seq Number: 80000042
  Checksum: 0xBC4A
  Length: 60
  Number of Links: 3

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 100.100.100.1
     (Link Data) Router Interface address: 100.100.100.1
      Number of TOS metrics: 0
       TOS 0 Metrics: 1</pre>

It’s prudent to note that I’m not showing all the Links advertise in such LSA’s, only the one’s relevant to this discussion. Now let’s take a look at **R2’s** own Router LSA.

<pre lang="plain">R2(config)#do sh ip ospf data router 200.0.0.2

                Router Link States (Area 100)

  LS age: 965
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 200.0.0.2
  Advertising Router: 200.0.0.2
  LS Seq Number: 80000040
  Checksum: 0xAC9F
  Length: 36
  Area Border Router
  Number of Links: 1

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 150.100.12.1
     (Link Data) Router Interface address: 150.100.12.2
      Number of TOS metrics: 0
       TOS 0 Metrics: 1</pre>

If we do simple math, of **BB1’s** advertised cost of one, plus **R1’s** advertised cost of one (the link connected to BB1), plus **R2’s** own cost of one to reach **R1**. It’s simple to see that if we look at **R2’s** generated Summary LSA into Area 0 for **BB1’s** loopback0 interface, that the cost will be three. Let’s take a look to confirm this;

<pre lang="plain">R2(config)#do sh ip ospf data summary 101.101.101.0

            OSPF Router with ID (200.0.0.2) (Process ID 1)

                Summary Net Link States (Area 0)

  LS age: 1480
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 101.101.101.0 (summary Network Number)
  Advertising Router: 200.0.0.2
  LS Seq Number: 80000021
  Checksum: 0xA15
  Length: 28
  Network Mask: /24
        TOS: 0  Metric: 3</pre>

How would the rest of the network see this then, let’s take a look to find out. Let’s examine **R5’s** LSA Database for the Summary LSA from **R2** and examine it.

**R5’s** OSPF Database

<pre lang="plain">R5#sh ip ospf data summ 101.101.101.0

            OSPF Router with ID (200.0.0.5) (Process ID 1)

                Summary Net Link States (Area 0)

  Routing Bit Set on this LSA
  LS age: 1108
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 101.101.101.0 (summary Network Number)
  Advertising Router: 200.0.0.2
  LS Seq Number: 80000027
  Checksum: 0xFD1B
  Length: 28
  Network Mask: /24
        TOS: 0  Metric: 3

                Summary Net Link States (Area 567)

  LS age: 1227
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 101.101.101.0 (summary Network Number)
  Advertising Router: 200.0.0.5
  LS Seq Number: 80000027
  Checksum: 0xF51F
  Length: 28
  Network Mask: /24
        TOS: 0  Metric: 4</pre>

We can see a Summary Net Link LSA from **R2** in Area 0, and one from **R5** itself to Area 567. The one received from R2 has a metric of three as expected, since that’s R2’s metric for that prefix. And the one from R5 has a metric of four; but why four? Let’s examine the cost of the frame relay network (configured as point-to-multipoint) from R5 to R2.

<pre lang="plain">R5#sh ip route 150.100.100.2
Routing entry for 150.100.100.2/32
  Known via "ospf 1", distance 110, metric 1, type intra area
  Last update from 150.100.100.2 on Serial0/0, 21:40:03 ago
  Routing Descriptor Blocks:
  * 150.100.100.2, from 200.0.0.2, 21:40:03 ago, via Serial0/0
      Route metric is 1, traffic share count is 1

R5#sh ip route 101.101.101.0
Routing entry for 101.101.101.0/24
  Known via "ospf 1", distance 110, metric 4, type inter area
  Last update from 150.100.100.2 on Serial0/0, 21:42:12 ago
  Routing Descriptor Blocks:
  * 150.100.100.2, from 200.0.0.2, 21:42:12 ago, via Serial0/0
      Route metric is 4, traffic share count is 1</pre>

**R5** simply adds the cost of the link to R2 in its own version of LSA Type-3 into area 567 and passes it on.

**R7’s** LSA Database

<pre lang="plain">R7(config)#do sh ip ospf data summ 101.101.101.0

            OSPF Router with ID (200.0.0.7) (Process ID 1)

                Summary Net Link States (Area 567)

  Routing Bit Set on this LSA
  LS age: 1776
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 101.101.101.0 (summary Network Number)
  Advertising Router: 200.0.0.5
  LS Seq Number: 80000027
  Checksum: 0xF51F
  Length: 28
  Network Mask: /24
        TOS: 0  Metric: 4</pre>

Again R7 upon receipt of the LSA with a metric of four from R5, simply adds its own cost of one inbound to R5 and set’s it’s metric to five.

<pre lang="plain">R7(config)#do sh run int f0/0
Building configuration...

Current configuration : 113 bytes
!
interface FastEthernet0/0
 ip address 150.100.220.7 255.255.255.0
 ip ospf cost 1
 speed 100
 full-duplex
end

R7(config)#do sh ip route 101.101.101.0
Routing entry for 101.101.101.0/24
  Known via "ospf 1", distance 110, metric 5, type inter area
  Last update from 150.100.220.5 on FastEthernet0/0, 21:44:28 ago
  Routing Descriptor Blocks:
  * 150.100.220.5, from 200.0.0.5, 21:44:28 ago, via FastEthernet0/0
      Route metric is 5, traffic share count is 1</pre>

LSA Type-3 or Network Summary LSA’s are flooded only within an Area, and each ABR forms its own version of the LSA and advertises that along. Since any area needs to cross the backbone area to reach any other area by default. It’s simply stated that Summary LSAs are only propagated from the backbone area into other areas. Or from any other single are into the backbone area zero.

Again, this was a bit longer than I’m used to writing :), hopefully helpful and I’m sure I’ll need to revisit and correct some grammatical or spelling mistakes. Or you can simply point them out and I’ll correct them as I see them.

 [1]: http://inetpy.com/assets/images/7946942c53ea1113b14c25472599014a.png
 [2]: http://inetpy.com/assets/images/24e635c09a840306a4625f7bb4eb28fb.png