---
title: 'Joining IPv6 Islands over an IPv4 Internet DMVPN'
author: yandy
layout: post
permalink: /2011/04/joining-ipv6-over-ipv4/
dsq_thread_id:
  - 2479751008
categories:
  - Networking
tags:
  - dmvpn
  - ipv4
  - ipv6
  - ipv6 over ipv4
  - network design
  - cisco
  - cisco dmvpn
---
# The Need to Run IPv6 {#theneedtorunipv6}

Well this one really doesn’t need much of an explanation anymore, IPv6 is here and IPv4 has been here for a long time. In most networks, the two must co-exist side by side or one on top of the other for a while, until the time where IPv6 becomes the only one. Hear that?; “It’s the sound of inevitability Mr. Anderson”. Whether you like to think about IPv6 or not, it’s here and it’s here to stay. So continuing on from a previous post on my own IPv6 series, we’re going to look at how to get disparate IPv6 sites over an IPv4 only Internet (the same can apply for an IPv4 only Backbone).

<!--more-->

# What this is not {#whatthisisnot}

It is not a primer on DMVPNs and how they work, <a href="https://twitter.com/#!/plapukhov" target="blank">Petr Lapukhov</a> has two great articles that go into some depth on how these work. The \[first one\]\[dmpvn1\] explaining Phase-I and Phase-II, the <a href="http://blog.ine.com/2008/12/23/dmvpn-phase-3/" target="blank">second one</a> explaining Phase-III of these technologies. Also, <a href="https://twitter.com/#!/ioshints" target="blank">Ivan Pepelnjak</a> has another really good <a href="http://www.ioshints.info/DMVPN:_From_Basics_to_Scalable_Networks" target="blank">webinar</a> dedicated to DMVPNs and he goes over allot of architecture decisions (I believe is only available as a recording now). And an <a href="http://www.ioshints.info/DMVPN_New_Features" target="blank">upcoming</a> one, that has few new features in the DMVPN world, and I’m sure like all the others, it’ll be great to attend.

<!--more-->

# Constraints in IPv6 Internet {#constraintsinipv6internet}

It’s quit simple, some providers have been slower to adopt IPv6 than many enterprises; reasons vary from:

  * network changes and outages
  * code upgrades to support dual-stack
  * new equipment that may need to be purchased
  * simply lack of understanding the need yet
  * demand not as high (yet)
  * …

So if you’ve made significant investment in a certain provider, that may not have native IPv6 transport services for you. Finding another, or making a lump investment may not be the best business case, or technical case. If you’re ready to deploy, don’t have private leased lines everywhere, or depend on MPLS services for inter-office (WAN) connections. And, these happen to be IPv4 only, luckily there’s a solution (bandaid) to that, with new IPv6 capable DMVPNs you can get going without depending on much but your own skill, and hardware.

# IPv6 DMVPN over IPv4 Transport {#ipv6dmvpnoveripv4transport}

This can provide “temporary” IPv6 over an IPv4 without the need for any 6-4 NAT (at within your routing domain). It’s at least a step towards IPv6 in production or at the very least a pseudo testing environment for IPv6 applications.

**You need**

  * End-to-end IPv4 connectivity (duh!)
  * Transport **HAS** to be IPv4
  * No native IPv6 transport to carry DMVPN traffic
  * Means: 
      * all NBMA mappings need to be to an IPv4 address
  * Other than that, same rules seem to apply
  * Phase-I, Phase-II, and Phase-III all work (as tested)

# The Topology {#thetopology}

So this is the topology we’ll be using **C1** and **C2** are our hubs. I’m using redundant Hubs with Dual tunnels, this just means spokes have two tunnels one to C1 and the other to C2, using routing metrics to prefer traffic. Along the way, I configured Phase-I and Phase-II for testing, and finally ended up at Phase-III which is the current state. Why? because it works, and I like it that way. Using, IPv6 EIGRP to route all the IPv6 traffic across the tunnels.

# Diagram {#diagram}

[<img alt="Diagram" src="{{ site.url }}/assets/images/IPv6-DMVPN-Topology.png" width="466" height="221" />][1]

# Getting Started {#gettingstarted}

As mentioned, C1 and C2 are Hub routers, where C1 is the primary and C2 is our secondary hub. All this is manipulated through route metrics. Again, EIGRP is the routing protocol for IPv6 tunnel traffic, and BGP is for our simulated Internet connections to pass our IPv4 WAN connections, used for (NBMA Mappings).

<pre lang="plain">C1#ping 10.0.0.2 sou lo0
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.2, timeout is 2 seconds:
Packet sent with a source address of 10.0.0.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/5/8 ms
C1#ping 10.0.0.5 sou lo0
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.5, timeout is 2 seconds:
Packet sent with a source address of 10.0.0.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/8/8 ms
C1#ping 10.0.0.6 sou lo0
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.6, timeout is 2 seconds:
Packet sent with a source address of 10.0.0.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/7/12 ms</pre>

  * 10.0.0.2 is C2
  * 10.0.0.5 is R2
  * 10.0.0.6 is R3
  * 10.0.0.1 is C1 the one doing the pings

All routers have Serial1/0 connected to the internet as PPP links and carrying all transport traffic across.

# C1’s Perspective {#c1sperspective}

Let’s look at some relevant configuration for C1 and it’s view of the network.

<pre lang="plain">interface Loopback0
 ip address 10.0.0.1 255.255.255.255
!
interface Tunnel0
 bandwidth 5000
 ip mtu 1400
 ip tcp adjust-mss 1360
 delay 10
 ipv6 address 2001:192:168::1/64
 ipv6 mtu 1400
 ipv6 eigrp 65001
 ipv6 summary-address eigrp 65001 ::/0 leak-map LEAK
 ipv6 nhrp authentication IPv6A
 ipv6 nhrp map multicast dynamic
 ipv6 nhrp network-id 13456
 ipv6 nhrp redirect
 tunnel source Loopback0
 tunnel mode gre multipoint
 tunnel key 13456
 tunnel protection ipsec profile DMVPN shared
!
interface FastEthernet0/0
 ipv6 address 2001:10:1:13::1/64
 ipv6 eigrp 65001
!</pre>

So Loopback0 is our real address we’ll use to carry our IPv6 Tunnel traffic, and to map our multicast at the Spokes. Yes, it is a /32 and advertised into BGP, no providers will never accept a /32 into their network from a customer, but it’s just a simulated internet of **one** router ;). Our tunnel is configured to be part of EIGRP as well as Fa0/0 which is really a segment into itself, nothing connected, but to simulate a separate network other than the Tunnel, I just think loopbacks are overused now for this purpose :).

If you’re already familiar with DMVPNs which you should be, there’s a few links at the top of the post. Then you’ll notice that all commands on the Hub side are pretty much identical (IPSec profile config not shown). I’ll post the full router configurations at the bottom of the post for anyone that wants to try them.

# C1’s EIGRP Participation {#c1seigrpparticipation}

Here’s how the above looks like in the EIGRP topology of greater things.

<pre lang="plain">C1#sh ipv6 eigrp int
EIGRP-IPv6 Interfaces for AS(65001)
                        Xmit Queue   Mean   Pacing Time   Multicast    Pending
Interface        Peers  Un/Reliable  SRTT   Un/Reliable   Flow Timer   Routes
Tu0                2        0/0        25       0/52          84           0
Fa0/0              1        0/0        14       0/1           68           0

C1#sh ipv6 eigrp neigh
EIGRP-IPv6 Neighbors for AS(65001)
H   Address                 Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                            (sec)         (ms)       Cnt Num
2   Link-local address:     Tu0               12 00:09:25   21   312  0  5
    FE80::C806:47FF:FEA3:8
1   Link-local address:     Tu0               13 00:09:33   29   312  0  5
    FE80::C807:47FF:FEA3:8
0   Link-local address:     Fa0/0             14 00:10:49   14   200  0  6
    FE80::C801:47FF:FEB1:8</pre>

We have two neighbors through Tunnel0 and one through the F0/0 interface, that’s just C2 on the other side of things. C2’s configuration is pretty much identical, except it’s tunnel has a delay of 20 and not 10, with different IPv6 subnets.

And here’s the IPv6 routing table for C1;

<pre lang="plain">C1#sh ipv6 route eigrp
IPv6 Routing Table - default - 9 entries

D   ::/0 [90/25856]
     via Null0, directly connected
D   2001:10:1:22::/64 [90/517120]
     via FE80::C806:47FF:FEA3:8, Tunnel0
D   2001:10:1:23::/64 [90/517120]
     via FE80::C807:47FF:FEA3:8, Tunnel0
D   2001:192:168:1::/64 [90/517376]
     via FE80::C801:47FF:FEB1:8, FastEthernet0/0</pre>

The ::/0 is a summary route created by C1 itself and advertised to the spokes, with Phase-III there’s no need to have full routing visibility at the spokes, I think that’s pretty cool. the :23::/64 is the ethernet segment on R3 and the :22::/64 is the ethernet segment on R2. The 2001:192:168:1::/64 is C2’s tunnel subnet, on C1 it’s 2001:192:168::/64 or for clarity 2001:192:168:0::/64.

# R2 and R3’s Perspectives {#r2andr3sperspectives}

Let’s take a look at R2 an R3’s configuration and view into the Tunnel cloud.

# R2 Configuration {#r2configuration}

<pre lang="plain">interface Loopback0
 ip address 10.0.0.5 255.255.255.255
!
interface Tunnel0
 bandwidth 2000
 no ip address
 no ip redirects
 ip mtu 1400
 ip tcp adjust-mss 1360
 delay 10
 ipv6 address 2001:192:168::5/64
 ipv6 mtu 1400
 ipv6 eigrp 65001
 ipv6 nhrp authentication IPv6A
 ipv6 nhrp map 2001:192:168::1/128 10.0.0.1
 ipv6 nhrp map multicast 10.0.0.1
 ipv6 nhrp network-id 13456
 ipv6 nhrp nhs 2001:192:168::1
 ipv6 nhrp shortcut
 tunnel source Loopback0
 tunnel mode gre multipoint
 tunnel key 13456
 tunnel protection ipsec profile DMVPN shared
!
interface Tunnel1
 bandwidth 2000
 no ip address
 no ip redirects
 ip mtu 1400
 ip tcp adjust-mss 1360
 delay 20
 ipv6 address 2001:192:168:1::5/64
 ipv6 mtu 1400
 ipv6 eigrp 65001
 ipv6 nhrp authentication IPv6B
 ipv6 nhrp map 2001:192:168:1::1/128 10.0.0.2
 ipv6 nhrp map multicast 10.0.0.2
 ipv6 nhrp network-id 23456
 ipv6 nhrp nhs 2001:192:168:1::1
 ipv6 nhrp shortcut
 tunnel source Loopback0
 tunnel mode gre multipoint
 tunnel key 23456
 tunnel protection ipsec profile DMVPN shared
!
interface FastEthernet0/0
 ipv6 address 2001:10:1:22::1/64
 ipv6 eigrp 65001
!</pre>

Two tunnels are created, one leading to C1 and the other to C2, C1 being preferred due to the routing metric, delay is 10 and the tunnel to C2 is 20. Other than that, R2’s and R3’s configuration is pretty much identical, except for addressing. Also take note, all static IPv6 logical addresses to NBMA &#8220;real&#8221; addresses map to the IPv4 interface of the tunnel source on the Hub and not an IPv6 address. This is the constraint that all transport is done over IPv4, there&#8217;s no IPv6 native transport for DMVPN yet.

# R2’s IPv6 Routing {#r2sipv6routing}

<pre lang="plain">R2#sh ipv6 eigrp neigh
EIGRP-IPv6 Neighbors for AS(65001)
H   Address                 Interface       Hold Uptime   SRTT   RTO  Q  Seq
                                            (sec)         (ms)       Cnt Num
2   Link-local address:     Tu1               12 01:55:15   41   786  0  10
    FE80::C801:47FF:FEB1:8
1   Link-local address:     Tu0               11 01:55:33   40  1179  0  12
    FE80::C800:47FF:FEB1:8

R2#sh ipv6 route eigrp

D   ::/0 [90/1282816]
     via FE80::C800:47FF:FEB1:8, Tunnel0
D   2001:10:1:13::/64 [90/1282816]
     via FE80::C800:47FF:FEB1:8, Tunnel0</pre>

We can see the default summary learned from the Hub, along with the leaked route directly connected to the hub. That was just for fun, no need to have that in there, but doesn’t really hurt either. **R3’s** routing table will look identical to this.

# Time to Test {#timetotest}

Let’s do a couple of pings and traces and make sure that our IPv6 traffic is flowing.

#### Pings and Traces from R2 {#pingsandtracesfromr2}

<pre lang="plain">R2#trace          
Protocol [ip]: ipv6           
Target IPv6 address: 2001:10:1:23::1
Source address: 2001:10:1:22::1
…. 
Type escape sequence to abort.
Tracing the route to 2001:10:1:23::1

  1 2001:192:168::1 40 msec 12 msec 8 msec
  2 2001:192:168::6 52 msec 24 msec 64 msec</pre>

So our first attempt goes through C1 and then on to R3 as expected, no let’s see our second attempt;

<pre lang="plain">R2#trace          
Protocol [ip]: ipv6           
Target IPv6 address: 2001:10:1:23::1
Source address: 2001:10:1:22::1
Insert source routing header? [no]: 
Numeric display? [no]: 
Timeout in seconds [3]: 
Probe count [3]: 
Minimum Time to Live [1]: 
Maximum Time to Live [30]: 
Priority [0]: 
Port Number [0]: 
Type escape sequence to abort.
Tracing the route to 2001:10:1:23::1

  1 2001:192:168::6 20 msec 12 msec 8 msec</pre>

Just like any other good old DMVPN Phase-III once NHRP does it’s thing, we have direct spoke to spoke communication as well. This would’ve worked again, with any of the phases, I just happen to configure Phase-III for this scenario. I do encourage you to visit, read, and watch the links in the beginning of this post if you’re unfamiliar with DMVPNs. They’re a pretty cool tech, but also not with out it’s caveats.

# Summary {#summary}

So here’s one way to tie in IPv6 islands through an IPv4 transport only Internet or Backbone network. Could be used for “testing”, proof of concept or temporary solution in your enterprise. I will agree that it may not be the cleanest solution, DMVPNs are great, but also present a few hurdles once in a while. But, I’m a firm believer that it’s better to have options, even if not always pretty, than to not have any.

Next post I’ll cover how to do the same with **LISP**, which is quickly becoming one of my favorite new things. These are not production tests, only labs and proof of concept type of things, so make sure you test them before putting them in your network. Specially LISP, not many production ready LISP environments out there yet, but I have feeling that they’ll be a few soon.

 [1]: {{ site.url }}/images/assests/IPv6-DMVPN-Topology.png