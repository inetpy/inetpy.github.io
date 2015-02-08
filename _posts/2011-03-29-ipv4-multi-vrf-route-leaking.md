---
title: 'Common Services VRF MPLS and BGP'
author: yandy
layout: post
permalink: /2011/03/ipv4-multi-vrf-route-leaking
image: vrf_leak.gif
twitter_image: vrf_leak.gif
description: Run services to a Central site to all other sites, without each remote having visibility to other remote sites.
summary: One of the most common MPLS VPN topologies is the Common Services simply put, it provides the most control of Branch traffic and filtering. MPLS VPNs are among one of today’s favorite and for good reason, it’s mainly scalable and provides nice redundancy in the SP network. But it also has it’s uses in Enterprise Campus and WAN networks.
dsq_thread_id:
  - 2479801914
categories:
  - Uncategorized
tags:
  - bgp
  - mpls
  - mpls vpn
---
![](http://ipyandy.net/images/vrf_leak.gif)

One of the most common **MPLS VPN** topologies is the **Common Services** simply put, it provides the most control of Branch traffic and filtering. MPLS VPNs are among one of today’s favorite and for good reason, it’s mainly scalable and provides nice redundancy in the SP network. But it also has it’s uses in Enterprise Campus and WAN networks. <a class="external" style="border: dotted blue 1px;" title="Follow on Twitter" href="https://twitter.com/#!/ioshints/" target="blank">Ivan Pepelnjak</a> has an awesome <a class="external" style="border: dotted black 1px;" title="Enterprise MPLS VPN Deployment" href="http://www.ioshints.info/Enterprise_MPLS_VPN_Deployment/" target="blank">Webinar</a> that covers allot of what you need to know. This blog post is simply to illustrate configuration samples and how to leak routes between MPLS L3 VPNs. This can be useful for for filtering or non-filtering of traffic between branch sites. As it stands, it’s not a BGP or MPLS VPN primer so it’s expected you have some knowledge of the subject. I will include footnotes with some *read on your own* material covering these topics.

**Warning, this will be a long post** —> you’ve been warned! :)

# Working Topology {#workingtopology}

Here’s the topology I’ll be working with throughout the rest of this post.  
[<img id="topology" title="Topology" alt="Topology" src="{{ site.url }}/assets/images/fb9b6436fb559c502c14499964e41380.png" width="NaN" height="NaN" />][1]  
<!--more-->

# Scenario

We have a **Central Site** or **Corp HQ** where all branch offices connect to. Right now, everything’s pre-configured as a Common Services topology. Branch sites do not talk to each other, while **Central** can see all other branches. But for some reason, there’s a requirement to get Site-B and Site-C communicating directly through the network core. Having these requirements in mind let’s start to take a look at configurations for each site. For the multi-homed sites, such as **Central** and **Site-A** I’ll only show the configuration for one router. This is sufficient as the other router’s configuration is exactly the same, except for minor differences in **IP** addresses and such. All routers have serial links into the core, either as **PPP** or **Frame-Relay** links. They’re running *OSPF Area 0* and have *mpls ip* enabled with to propagate the *labels* through the core. This is done to minimize the number of *BGP* sessions and have a *BGP* less core, which is very common in *MPLS* enabled networks.

# Central Site PE Configuration of VRF {#hubsitepeconfigurationofvrf}

Let’s take a look at the **VRF** definition and see what it looks like.

<pre lang="“plain”">vrf definition central
 rd 65000:100
 !
 address-family ipv4
 route-target export 65000:100
 route-target import 65000:100
 route-target import 65000:102
 route-target import 65000:103
 route-target import 65000:104
 exit-address-family
!</pre>

The **route-distinguisher’s** *65000:100*, I chose this because of the BGP AS tying down all the sites, and *100* being the first VRF. It’s importing **route-targets** of *65000:100-104*, which is encompasses all the other sites, and exports *65000:100*. And here’s the **Central Site&#8217;s** assigned *VRF* interface for this segment.

<pre lang="“plain”">interface FastEthernet0/1.11
 vrf forwarding central
 encapsulation dot1Q 11
 ip address 10.100.11.1 255.255.255.0
!</pre>

Simple really, as long as you have some knowledge of *VRF* or *VRF Lite* this part should look easy enough. The CE to PE protocol used at the **Central** site’s *BGP*, simply because I wanted to get as many protocols involved in this post as possible. Site-A is running *OSPF* as a PE to CE protocol. Site-B is running EIGRP and Site-C is running RIP.

# Central PE VPNv4 BGP Config {#hubpevpnv4bgpconfig}

Now let’s take a look at the *Central* site’s BGP configuration, which is kinda long so I’ll dissect it little by little.

<pre lang="“plain”">router bgp 65000
 template peer-policy iBGP-RR
  route-reflector-client
 exit-peer-policy
 !
 template peer-session iBGP
  remote-as 65000
  update-source Loopback0
 exit-peer-session
 !</pre>

If you’re not familiar with *BGP peer templates* I highly suggest you do, they’re very cool. But I can’t go into details, other than they’re like *peer-groups* but better; now the rest of the configuration.

<pre lang="“plain”">no bgp default ipv4-unicast
 bgp log-neighbor-changes
 neighbor 172.16.0.2 inherit peer-session iBGP
 neighbor 172.16.1.1 inherit peer-session iBGP
 neighbor 172.16.1.2 inherit peer-session iBGP
 neighbor 172.16.3.3 inherit peer-session iBGP
 neighbor 172.16.4.4 inherit peer-session iBGP
 !
 address-family vpnv4
  neighbor 172.16.0.2 activate
  neighbor 172.16.0.2 send-community both
  neighbor 172.16.1.1 activate
  neighbor 172.16.1.1 send-community both
  neighbor 172.16.1.1 inherit peer-policy iBGP-RR
  neighbor 172.16.1.2 activate
  neighbor 172.16.1.2 send-community both
  neighbor 172.16.1.2 inherit peer-policy iBGP-RR
  neighbor 172.16.3.3 activate
  neighbor 172.16.3.3 send-community both
  neighbor 172.16.3.3 inherit peer-policy iBGP-RR
  neighbor 172.16.4.4 activate
  neighbor 172.16.4.4 send-community both
  neighbor 172.16.4.4 inherit peer-policy iBGP-RR
 exit-address-family
 !
 address-family ipv4 vrf central
  no synchronization
  neighbor 10.100.11.11 remote-as 65100
  neighbor 10.100.11.11 activate
 exit-address-family
!</pre>

This scenario does not use BGP to pass IPv4 addresses in the global table, so the address family has been removed from the config output. BGP’s only being used as a CE to PE protocol and o pass *VPNv4* routes across the *backbone* of the network. The **C1** router has a session to each branch and acts as the *route-reflector* for them. As well as a session to **C2** router which has a similar configuration. One very important thing to notice is, that, for each *iBGP* neighbor, we have to send an extended community. As you can see, under *address-family ipv4 vrf central* which is the last section in the configuration. The Central PE router has a *BGP* session to **CR1** which is the **Central Site* CE router; outlined below;

**C1 VRF Central BGP Neighbors**

<pre lang="“plain”">C1#sh bgp vpnv4 unicast vrf central summary
BGP router identifier 172.16.0.1, local AS number 65000
BGP table version is 18, main routing table version 18
8 network entries using 1152 bytes of memory
9 path entries using 468 bytes of memory
11/8 BGP path/bestpath attribute entries using 1452 bytes of memory
4 BGP rrinfo entries using 96 bytes of memory
1 BGP AS-PATH entries using 24 bytes of memory
6 BGP extended community entries using 628 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3820 total bytes of memory
BGP activity 15/0 prefixes, 27/1 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.100.11.11    4        65100    1175    1298       18    0    0 19:29:00        1</pre>

We can see that the neighbor’s up and we’re receiving **one** prefix from it. Let’s take a look to see what that prefix is;

**Central VRF VPNv4 Neighbor Routes**

<pre lang="plain">C1#sh bgp vpnv4 unicast vrf central neighbors 10.100.11.11 routes
BGP table version is 18, local router ID is 172.16.0.1
Status codes: s suppressed, d damped, h history, * valid, &gt; best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 65000:100 (default for vrf central)
*&gt; 192.168.1.0      10.100.11.11             0             0 65100 i

Total number of prefixes 1</pre>

We’re receiving the *192.168.1.0/24* and if we look further, there’s some attributes that can in understanding just what’s going on behind the scenes.

**BGP Table Entry 192.168.1.0**

<pre lang="plain">C1#sh bgp vpnv4 unicast vrf central 192.168.1.0
BGP routing table entry for 65000:100:192.168.1.0/24, version 2
Paths: (2 available, best #2, table central)
  Advertised to update-groups:
     2          3
  65100
    172.16.0.2 (metric 2) from 172.16.0.2 (172.16.0.2)
      Origin IGP, metric 0, localpref 100, valid, internal
      Extended Community: RT:65000:100
      mpls labels in/out 27/27
  65100
    10.100.11.11 from 10.100.11.11 (192.168.1.1)
      Origin IGP, metric 0, localpref 100, valid, external, best
      Extended Community: RT:65000:100
      mpls labels in/out 27/nolabel</pre>

We can see a couple of things right away

  * the prefix has been assigned and route-distinguisher (RD) of 65000:100
  * the route-target for it being RT:65000:100 as part of a BGP extended community  
    – hence the need to send extended communities to BGP VPNv4 neighbors
  * the incoming and outgoing labels, NULL and 27 respectively for the *best* path

**And here’s the routing table for the** ***Central Site*** **VRF**

<pre lang="plain">C1#sh ip route vrf central

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
B        10.3.3.0/24 [200/0] via 172.16.3.3, 20:24:09
B        10.4.4.0/24 [200/0] via 172.16.4.4, 20:24:09
C        10.100.11.0/24 is directly connected, FastEthernet0/1.11
L        10.100.11.1/32 is directly connected, FastEthernet0/1.11
B        10.100.112.0/24 [200/0] via 172.16.1.1, 20:24:25
B        10.100.122.0/24 [200/0] via 172.16.1.2, 20:24:10
B     192.168.1.0/24 [20/0] via 10.100.11.11, 20:24:26
B     192.168.2.0/24 [200/102] via 172.16.1.1, 20:24:25
B     192.168.3.0/24 [200/156160] via 172.16.3.3, 20:24:09
B     192.168.4.0/24 [200/1] via 172.16.4.4, 20:24:09</pre>

Other than the *Central VRF* routes, we have the routes for the other sites. Which we will look at in detail shortly.

# Central Site Router CR1 {#centralsiteroutercr1}

There’s not much here other than a simple routing protocol and interface IP configurations as usual

<pre lang="“plain”">interface Loopback0
 ip address 192.168.1.1 255.255.255.0
!
interface FastEthernet0/1.11
 encapsulation dot1Q 11
 ip address 10.100.11.11 255.255.255.0
!
interface FastEthernet0/1.12
 encapsulation dot1Q 12
 ip address 10.100.12.11 255.255.255.0
!
router bgp 65100
 no synchronization
 bgp log-neighbor-changes
 network 192.168.1.0
 neighbor 10.100.11.1 remote-as 65000
 neighbor 10.100.12.2 remote-as 65000
 no auto-summary
!</pre>

The loopback interface is being advertised into BGP which is the route we saw in the previous section. It also has two neighbors **C1** and **C2**, which we can see the output for.

<pre lang="plain">CR1#sh ip bgp summ
BGP router identifier 192.168.1.1, local AS number 65100
BGP table version is 9, main routing table version 9
8 network entries using 960 bytes of memory
15 path entries using 780 bytes of memory
3/2 BGP path/bestpath attribute entries using 372 bytes of memory
1 BGP AS-PATH entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
Bitfield cache entries: current 1 (at peak 1) using 32 bytes of memory
BGP using 2168 total bytes of memory
BGP activity 8/0 prefixes, 15/0 paths, scan interval 60 secs

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.100.11.1     4 65000    1364    1235        9    0    0 20:29:34        7
10.100.12.2     4 65000    1359    1235        9    0    0 20:29:38        7</pre>

**BGP RIB**

<pre lang="plain">CR1#sh ip bgp
BGP table version is 9, local router ID is 192.168.1.1
Status codes: s suppressed, d damped, h history, * valid, &gt; best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*  10.3.3.0/24      10.100.12.2                            0 65000 ?
*&gt;                  10.100.11.1                            0 65000 ?
*  10.4.4.0/24      10.100.12.2                            0 65000 ?
*&gt;                  10.100.11.1                            0 65000 ?
*  10.100.112.0/24  10.100.12.2                            0 65000 ?
*&gt;                  10.100.11.1                            0 65000 ?
*  10.100.122.0/24  10.100.12.2                            0 65000 ?
*&gt;                  10.100.11.1                            0 65000 ?
*&gt; 192.168.1.0      0.0.0.0                  0         32768 i
*  192.168.2.0      10.100.12.2                            0 65000 ?
*&gt;                  10.100.11.1                            0 65000 ?
*  192.168.3.0      10.100.12.2                            0 65000 ?
*&gt;                  10.100.11.1                            0 65000 ?
*  192.168.4.0      10.100.12.2                            0 65000 ?
*&gt;                  10.100.11.1                            0 65000 ?</pre>

And all the routes from all the other Sites, being that this is the **Central** site

**Here’s the complete routing table**

<pre lang="plain">CR1#sh ip route

Gateway of last resort is not set

B    192.168.4.0/24 [20/0] via 10.100.11.1, 20:28:13
     10.0.0.0/24 is subnetted, 6 subnets
B       10.4.4.0 [20/0] via 10.100.11.1, 20:28:13
B       10.3.3.0 [20/0] via 10.100.11.1, 20:28:13
B       10.100.122.0 [20/0] via 10.100.11.1, 20:28:29
B       10.100.112.0 [20/0] via 10.100.11.1, 20:28:29
C       10.100.12.0 is directly connected, FastEthernet0/1.12
C       10.100.11.0 is directly connected, FastEthernet0/1.11
C    192.168.1.0/24 is directly connected, Loopback0
B    192.168.2.0/24 [20/0] via 10.100.11.1, 20:28:29
B    192.168.3.0/24 [20/0] via 10.100.11.1, 20:28:13</pre>

Nothing special here, as all the magic is happening on the PE side of the backbone network.

# Site-A PE Router View {#site-aperouterview}

From the perspective of *Site-A* which is a complete isolated site, the only routes being taken in will be the *central site* prefixes. This can be clearly illustrated with the VRF Configuration.

<pre lang="“plain”">vrf definition Site-A
 rd 65000:102
 !
 address-family ipv4
 route-target export 65000:102
 route-target import 65000:100
 exit-address-family
!</pre>

This site is exporting it’s VPNv4 routes as 65000:102 but only importing 65000:100, which are the prefixes marked as *Central Site*. Let’s take a look at the VPNv4 BGP table to compare;

**R1 VPNv4 BGP Table**

<pre lang="plain">R1#sh bgp vpnv4 unicast all
BGP table version is 6, local router ID is 172.16.1.1
Status codes: s suppressed, d damped, h history, * valid, &gt; best, i - internal,
              r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 65000:100
*&gt;i192.168.1.0      172.16.0.1               0    100      0 65100 i
* i                 172.16.0.2               0    100      0 65100 i
Route Distinguisher: 65000:102 (default for vrf Site-A)
*&gt; 10.100.112.0/24  0.0.0.0                  0         32768 ?
*&gt; 10.100.122.0/24  10.100.112.12          102         32768 ?
*&gt;i192.168.1.0      172.16.0.1               0    100      0 65100 i
*&gt; 192.168.2.0      10.100.112.12          102         32768 ?</pre>

There’s a couple of things to note

  * only the prefixes marked with *65000:100* and *65000:102* are showing in the table,  
    this is shown by the *Route Distinguisher* output
  * the 10.100.X.X networks are the connected networks to the PE R1 router
  * the prefix 192.168.2.0 is the route being advertised by CR2 which is learned through OSPF  
    then redistributed into BGP.

**192.168.2.0/24 OSPF Route in VRF Site-A**

<pre lang="plain">R1#sh ip route vrf Site-A 192.168.2.0

Routing Table: Site-A
Routing entry for 192.168.2.0/24
  Known via "ospf 102", distance 110, metric 2, type intra area
  Redistributing via bgp 65000
  Advertised by bgp 65000 metric 102
  Last update from 10.100.112.12 on FastEthernet0/0.112, 21:15:09 ago
  Routing Descriptor Blocks:
  * 10.100.112.12, from 192.168.2.2, 21:15:09 ago, via FastEthernet0/0.112
      Route metric is 2, traffic share count is 1</pre>

**OSPF Configuration**

<pre lang="plain">router ospf 102 vrf Site-A
 log-adjacency-changes
 redistribute bgp 65000 metric 102 subnets
 network 10.100.112.0 0.0.0.255 area 0
!</pre>

**BGP Configuration**

<pre lang="plain">router bgp 65000
 template peer-session iBGP
  remote-as 65000
  update-source Loopback0
 exit-peer-session
 !
 no bgp default ipv4-unicast
 bgp log-neighbor-changes
 neighbor 172.16.0.1 inherit peer-session iBGP
 neighbor 172.16.0.2 inherit peer-session iBGP
 !
 address-family vpnv4
  neighbor 172.16.0.1 activate
  neighbor 172.16.0.1 send-community both
  neighbor 172.16.0.2 activate
  neighbor 172.16.0.2 send-community both
 exit-address-family
 !
 address-family ipv4 vrf Site-A
  no synchronization
  redistribute ospf 102 vrf Site-A metric 102
 exit-address-family
!</pre>

BGP is being redistributed into *OSPF VRF Site-A* and OSPF is being redistributed into BGP to the backbone. Other than that most of the configuration is similar to the **C1** configuration we saw earlier.

**And here’s the routing table for** ***VRF Site-A***

<pre lang="plain">R1#sh ip route vrf Site-A

Routing Table: Site-A
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, + - replicated route

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C        10.100.112.0/24 is directly connected, FastEthernet0/0.112
L        10.100.112.1/32 is directly connected, FastEthernet0/0.112
O        10.100.122.0/24
           [110/2] via 10.100.112.12, 20:39:47, FastEthernet0/0.112
B     192.168.1.0/24 [200/0] via 172.16.0.1, 20:39:19
O     192.168.2.0/24 [110/2] via 10.100.112.12, 20:39:47, FastEthernet0/0.112</pre>

Again, we can see that only the *local* routes and the prefixes imported from the central site are being put in the Site-A VRF routing table as well. If it does not go through the filtering in BGP with route-targets, it will not go into the routing table.

# Site-A VRF CE Router &#8211; CR2 {#site-avrfcerouter-cr2}

Let’s take a look at Site-A’s configuration options.

**Interface and OSPF Configuration**

<pre lang="“plain">interface Loopback0
 ip address 192.168.2.2 255.255.255.0
 ip ospf network point-to-point
 ip ospf 102 area 0
!
interface FastEthernet0/0.112
 encapsulation dot1Q 112
 ip address 10.100.112.12 255.255.255.0
 ip ospf priority 0
!
interface FastEthernet0/0.122
 encapsulation dot1Q 122
 ip address 10.100.122.12 255.255.255.0
 ip ospf priority 0
!
router ospf 102
 log-adjacency-changes
 network 10.100.0.0 0.0.255.255 area 0
 network 192.168.2.0 0.0.0.255 area 0
!</pre>

Much like the **central site** nothing to complicated, simple routing and interfaces, no need for this CE to be VPN aware at all. The magic once again happens within PE->PE and the backbone network.

Let’s take a look at the routing table here just for giggles and to say all basis were covered.

<pre lang="plain">CR2#sh ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/24 is subnetted, 2 subnets
C       10.100.122.0 is directly connected, FastEthernet0/0.122
C       10.100.112.0 is directly connected, FastEthernet0/0.112
O E2 192.168.1.0/24 [110/102] via 10.100.122.2, 20:41:39, FastEthernet0/0.122
                    [110/102] via 10.100.112.1, 20:41:55, FastEthernet0/0.112
C    192.168.2.0/24 is directly connected, Loopback0</pre>

The route from the *Central Site* shows up as OSPF E2 as expected, since it’s being redistributed into OSPF from BGP. And we can see that a ping sourced from the loopback0 to the CR1 loopback0 interface works.

<pre lang="plain">CR2#ping 192.168.1.1 source 192.168.2.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.2.2
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/14/24 ms</pre>

# Site-B PE Router R3 {#site-bperouterr3}

Only relevant information to leaking the routes will be shown from now on, it’s not a primer, but it started to get overly long. Lets take a look at the **VRF** configuration to see what route targets we’ll be using.

<pre lang="plain">vrf definition Site-B
 rd 65000:103
 !
 address-family ipv4
 route-target export 65000:103
 route-target import 65000:100
 route-target import 65000:104
 exit-address-family
!</pre>

R3’s exporting 65000:103 and importing 65000:100 (central site) and 65000:104 which is Site-C. To make sure that these prefixes are being imported correctly, let’s take a look at the BGP VPNv4 tables.

# Site-B CE Router &#8211; CR3 {#site-bcerouter-cr3}

The configuration for this on is much of the same as the others, except it’s running EIGRP. Let’s take a look at the routing table and see what we have.

<pre lang="plain">CR3#sh ip route
…. SOME OUTPUT REMOVED….

Gateway of last resort is not set

D EX 192.168.4.0/24 [170/853336576] via 10.3.3.3, 17:38:58, FastEthernet0/0
D EX 192.168.1.0/24 [170/853336576] via 10.3.3.3, 22:15:28, FastEthernet0/0</pre>

Both expected routes are there, 192.168.1.0/24 (central site) and 192.168.4.0 (Site-C). Now let’s try to communicate and see what happens.

<pre lang="plain">CR3#ping 192.168.1.1 source 192.168.3.3

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.3.3
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/9/12 ms
CR3#ping 192.168.4.4 source 192.168.3.3

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.4.4, timeout is 2 seconds:
Packet sent with a source address of 192.168.3.3
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/12/16 ms</pre>

A ping to both sites worked, letting us know that prefixes are being passed as expected. Now let’s try to see if we can ping Site-A (we’re not supposed to be able to), and see what happens.

<pre lang="plain">CR3#ping 192.168.2.2 source 192.168.3.3

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.2, timeout is 2 seconds:
Packet sent with a source address of 192.168.3.3
.....
Success rate is 0 percent (0/5)</pre>

Failure tells us the correct filtering is in place, and we’re all good to go.

# Summary {#summary}

Putting Site-C in this post would be too redundant, and it’ll take two hours just to read this post (no one wants to do that). Basically the configuration is much of the same, but exporting 65000:104 and importing Site-B and central RT’s. I warned you in the beginning, this was a long post, longest by far I think. But I think it has some useful information, and that’s what matters :).

MPLS L3 VPNs are very flexible, allow for clean designs with out much of access-list controls. We all know access-lists can get messy really quick, and if not maintained, it can be more of a headache than do any good. Filtering three sites with access-lists is not a big deal; but what happens when you get into the hundreds of sites? Then maintaining those can be very cumbersome. Even though it was long, I hope you got something out of it. I plan to do another similar (shorter) version for IPv6, shorter because most of the basics were covered here.

# Recommended Reading {#recommendedreading}

  * <a class="external" style="border: dotted blue 1px;" title="MPLS and VPN Architectures" href="http://www.ciscopress.com/bookstore/product.asp?isbn=1587050021" target="blank">MPLS and VPN Architectures</a>
  * <a class="external" style="border: dotted blue 1px;" title="Internet Routing Architectures" href="http://www.ciscopress.com/bookstore/product.asp?isbn=157870233X/" target="blank">Internet Routing Architectures, Second Ed</a>
  * <a class="external" style="border: dotted blue 1px;" title="Routing TCP/IP Vol 1" href="http://www.ciscopress.com/bookstore/product.asp?isbn=1587052024/" target="blank">Routing TCP/IP Vol 1</a>
  * <a class="external" style="border: dotted black 1px;" title="ioshints" href="http://www.ioshints.info/" target="blank">ioshints</a>

<span style="color: #ff0000;"><strong>Edit:</strong></span>

Thanks to <a href="https://twitter.com/#!/ioshints/" target="blank">Ivan</a>

[1]: {{ site.url }}/assets/images/fb9b6436fb559c502c14499964e41380.png