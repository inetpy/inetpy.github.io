---
title: '{LISP} IPv6 over IPv4 Transition'
author: Yandy
layout: post
banner_image: Lisp-logo.jpg
image: lisp.jpg
twitter_image: lisp.jpg
permalink: 2011/04/lisp-ipv6-over-ipv4-transition/
description: Tunnel IPv6 over IPv4 using LISP
summary: First an foremost, I want to thank Ivan Pepelnjakof ipspace.net for taking the time to read, correct and validate these concepts. Taking time out of his busy schedule to make sure this information is as accurate as possible.
dsq_thread_id:
  - 2479721805
categories:
  - Networking
tags:
  - ipv6
  - ipv6-over-ipv4
  - ipv6-transition
  - lisp
---
First an foremost, I want to thank <a href="https://twitter.com/#!/ioshints" target="blank">Ivan Pepelnjak</a> of <a href="http://www.ipspace.net" target="blank">ipspace.net</a> for taking the time to read, correct and validate these concepts. Taking time out of his busy schedule to make sure this information is as accurate as possible.

<!--more-->

# Post Content {#postcontent}

Yes, yet another post on IPv6 and how to transition into it, while keeping intact your IPv4 infrastructure. This may get old soon, or not, but I’m using this topic to also present a few cool features of **LISP** (Locator/ID Separation Protocol). If you haven’t heard of LISP by now, go visit <a href="http://blog.fryguy.net/2011/04/07/lisp-locator-identifier-separation-protocol-say-what/" target="blank">Jeff Fry’s blog</a> where he covers in some detail the workings of LISP with working examples. I’m concentrating more on using this to pass IPv6 traffic through an IPv4 only network, in other words, tying in disparate IPv6 islands over an IPv4 internet or Backbone network.

# LISP Overview {#lispoverview}

If you haven’t gone over to Jeff’s Blog (linked above), here’s a brief description of LISP for the lazy. At it’s root LISP is a very simple thing. It adds another addressing semantics to an IP packet, much like MPLS adds labels, well LISP adds what it’s called an RLOC (Routing Locator) which you can see as the *label*. Basically means, this is where the IP/IPv6 address you’re looking for is at, all routers along the path just need to route to this outer IP address (Label analogy). Once the packet gets to this *Location* the LISP header is removed and sent on as a normal IP packet. Much like MPLS the rest of the network is unaware of what’s going on, and they’re just making per-hop decisions on the Location address (RLOCs), in MPLS terms Labels, and not the END destination. The Non-LISP side of the network, needs no changes or be aware of anything being changed either.

<!--more-->

#### Brief description of LISP Terms and Actions {#briefdescriptionoflisptermsandactions}

  * **RLOC** (Routing Locator), another fancy name for where the ultimate destination resides in, this is your routable address through the Internet or Core network.
  * You have an **EID**, fancy name for End-host, which resides on the Non-LISP side of a LISP site. These typically are locally routable addresses within the *site*, site could be an office, HQ, Campus Network, Datacenter, and so on. So your End-hosts are your servers, switches, routers, any networkable device (any device with an IPv4 or IPv6 address). This is basically a local address, that the internet or core network need not know about.
  * **EID** is Fixed (address never changes), RLOC is not, there could be multiple ways of getting to the host
  * There’s this concept of **ITR** and **ETR**, Ingress Tunnel Router and Egress Tunnel Router respectively 
      * An **ITR** receives non-LISP packets and “encapsulates” them (if that’s the correct term) or puts a LISP Header if the Destination is in another LISP site, (essentially in mPLS terms, a label ingress router), only in LISP this router imposes a LISP header
      * An **ETR** receives the **LISP** packet and removes the **LISP** **Header**, sends it on it’s way to the **EID** (end-host)
  * **ETRs** register their EID-to-RLOC mappings with the **Map Server** 
      * **xTR** = Either ETR+ITR
  * **Map Server (MS)**, these router(s) serve as the central database for **EID**-**prefix** to **RLOC** mappings.
  * The **Map Resolver (MR)** propagates mapping-requests to the **MS**, which in turn propagates the request to the **ETR**
  * **MS** and **MR** don’t have to be routers, they can be any form of generic servers. They are, however, implemented on numerous Cisco IOS/NX-OS platforms. Our MS/MR for this post is a Cisco router running IOS 15.1(4)M.
  * **ETR** can cache the return path to the **ITR**
  * Think of the **Map Server** / **Map Resolver** as a form of DNS, they propagate location information, only for IP addresses
  * The **LISP** **Map-cache** is a dynamic, short (usually) temporary database of on-demand** EID-prefixes** in other LISP sites. Built from dynamic communication with **MS/MR *\*and other \**xTRs**.
  * The validity of entries in **LISP** map-cache is specified with TTL fields (like DNS)
  * The LISP Database it’s the **xTRs** own local database of **EIDs**, that this **ETR** is responsible for **EID-to-RLOC** mappings.

LISP control packets are UDP encapsulated and with a random generated source port, and a destination port of (UDP:4342) which is the LISP data port. While, LISP data packets are also UDP encapsulated with a random generated Source Port (>= 1024), the exact algorithm is described in the LISP draft standard write up, the destination port is the LISP Data port (UDP:4341).

# Working Topology {#workingtopology}

We have the very similar topology that I’ve been using lately for just about everything (thanks to Ivan Pepelnjak). Last post for DMVPN I was using a simulated “Internet” here, it’s a simulated “Core” network. Same things apply really, as long as what you choose to be your *LISP Map Server/Resolver” is/are publicly visible over the internet, and or within your network.

Whether LISP gains global acceptance or not (I think it should and will), it’s quickly becoming a favorite toy. That said, it’s extremely new and probably untested, although a few groups have been testing it for sometime. Wide global enterprise or provider testing really isn’t main stream yet. Warning! Lab this sh*t up before you try it in your network :)

# Diagram {#diagram}

[<img id="lispdiagram" title="LISP Diagram" alt="LISP Diagram" src="{{ site.url }}/assets/images/LISP_IPv6_IPv4.png" />][1]

# Topology Summary {#topologysummary}

  * C1, R2 and R3 are xTR routers, meaning either ITR or ETR
  * C4, RB2, and RB3 are normal routers/hosts just running EIGRP for both IPv6 and IPv4
  * OSPF is used in the Core for RLOC reachability
  * MS_MR is our Map Server and Map Resolver for the network 
      * It’ll have knowledge of all LISP Sites, and our xTRs will communicate to register their LISP prefixes and resolve others

Although I’m concentrating on running IPv6 over an IPv4 network, the configurations are there for IPv4 only as well. I was just testing multiple functionality, and the configurations at the end of the post has both. Essentially, one of the cool things about LISP is that you can do IPv4 only, IPv6 only, IPv4 over IPv6, or IPv6 over IPv4 or any combination. I’m trying to get my hands on some LISP capable Nexus 7000s to test out the VM IP Mobility stuff if it’s there, more research on the go.

# The Nitty Gritty {#thenittygritty}

Now to the fun part of all this, theory’s great, and it helps to understand it. But seen it in practice and playing with the tech is what it’s all about. The C1, C4 part of the network will be our MainSite, R2, RB2 will be SiteTwo and R3, RB3 will be SiteThree. This is just naming convention and nothing to do with anything else.

Why don’t we examine a bit of where our central database for EID-prefix to RLOC mapping is located. Which would be our MS_MR router (Map Server, Map Resolver). Although this is a router, it is stated that it can be any networkable device capable of running LISP Mapping services.

#### Map Server / Resolver {#mapserverresolver}

{% highlight text %}
MS_MR#sh run | sec router lisp
router lisp
 site MainSite
  description C1_C2_C4
  authentication-key LISP
  eid-prefix 2001:10:1:14::/64
  eid-prefix 2001:10:1:114::/64
 !
 site SiteThree
  description R3_RB3
  authentication-key LISP
  eid-prefix 2001:10:1:23::/64
  eid-prefix 2001:10:1:123::/64
 !
 site SiteTwo
  description R2_RB2
  authentication-key LISP
  eid-prefix 2001:10:1:22::/64
  eid-prefix 2001:10:1:122::/64
 ! 
 ipv6 map-server
 ipv6 map-resolver
 ipv6 alt-vrf LISP
 {% endhighlight %}
 
Only the IPv6 relevant configuration and information will be shown throughout the rest of the blog post. This is the relevant Map Server / Map Resolver configuration. As you can see, there’s really not much to the configuration itself;

  * The sites are defined
  * The EID-prefixes that belong to those sites are set
  * An authentication Key is set to authenticate Map Register Messages from ITRs
  * The MS/MR is configured as such

Once the xTRs have registered with the Map Server all the EID-Prefixes within the site, the Map Server will have a table of the Sites, EID-to-RLOC mappings.

<pre lang="shell">MS_MR#sh lisp site 
LISP Site Registration Information

Site Name      Last      Up   Who Last             EID Prefix
               Register       Registered           
MainSite       00:00:15  yes  10.0.1.1             10.1.14.0/24
               00:00:15  yes  10.0.1.1             10.1.114.0/24
               00:00:11  yes  10.0.1.5             2001:10:1:14::/64
               00:00:11  yes  10.0.1.5             2001:10:1:114::/64
SiteThree      00:00:00  yes  10.0.1.13            10.1.23.0/24
               00:00:00  yes  10.0.1.13            10.1.123.0/24
               00:00:40  yes  10.0.1.13            2001:10:1:23::/64
               00:00:40  yes  10.0.1.13            2001:10:1:123::/64
SiteTwo        00:00:11  yes  10.0.1.9             10.1.22.0/24
               00:00:11  yes  10.0.1.9             10.1.122.0/24
               00:00:39  yes  10.0.1.9             2001:10:1:22::/64
               00:00:39  yes  10.0.1.9             2001:10:1:122::/64</pre>

The table’s fairly easy to decipher, although I probably would’ve put EID Prefix before “Who Last Registered”. Either way, you can see the Site Name, last time it was updated (which happens to be every minute) if you watch the timers. There’s UP status, usually a good thing if you see “yes” in this column, Who Last Registered is the last ETR to register this EID-prefix. And, last but not least you have the EID-prefix itself. I find it easier to read the table from right to left.

  * The EID-prefix
  * Who Registered it (ETR)
  * If it’s up or not?
  * How long it’s been registered for
  * And the Site that it belongs to

# RB2 Pings C4 {#rb2pingsc4}

Now let’s begin some actual end-to-end connectivity testing. Initially we’ll be doing ICMP testing from **RB2** in **SiteTwo** to **C4** in **MainSite**. This includes Ping and Trace-route tests to see what the can be gleaned from this information.

#### First Ping Test from RB2 {#firstpingtestfromrb2}

<pre lang="shell">RB2#ping 2001:10:1:114::14 sou lo0 re 20
Type escape sequence to abort.
Sending 20, 100-byte ICMP Echos to 2001:10:1:114::14, timeout is 2 seconds:
Packet sent with a source address of 2001:10:1:122::22
..!!!!!!!!!!!!!!!!!!
Success rate is 90 percent (18/20), round-trip min/avg/max = 12/14/20 ms</pre>

Very simple test, we should all know what these commands do, but just in case;

  * Ping an IPv6 address of 2001:10:1:114::14
  * Sourcing those packets from Loopback0 interface (2001:10:1:122::22)
  * And sending a total of 20 ICMP packets

We thing that may stand out right away is that it missed two ping packets, with a default of two second timeout. This means that it took four seconds to find it’s way through our network. In constant testing, I’ve noticed that it misses anywhere from one to four packets. Though, once it has all the information, even after clearing the map-cache table, the most it ever misses is one packet. In a production environment, it may be less or more, delay, bandwidth an QoS, CPU Load, all these things may come into play here.

Lets examine what’s going on under the hood as the LISP AFI Map-cache is built.

#### R2 IPv6 RIB {#r2ipv6rib}

<pre lang="shell">R2#sh ipv6 route

IPv6 Routing Table - default - 4 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, HA - Home Agent, MR - Mobile Router, R - RIP
       I1 - ISIS L1, I2 - ISIS L2, IA - ISIS interarea, IS - ISIS summary
       D - EIGRP, EX - EIGRP external, ND - Neighbor Discovery, l - LISP
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2
C   2001:10:1:22::/64 [0/0]
     via FastEthernet0/0, directly connected
L   2001:10:1:22::1/128 [0/0]
     via FastEthernet0/0, receive
D   2001:10:1:122::/64 [90/156160]
     via FE80::C80A:47FF:FE95:8, FastEthernet0/0
L   FF00::/8 [0/0]
     via Null0, receive</pre>

Once R2 receives the first ICMP Echo Request packet destined for C4’s Loopback0 interface of 2001:10:1:114::14. R2 checks it’s routing table and does not find a route to that subnet in there. As you can see there’s just a the Local subnet connecting to RB2 and an EIGRP route to RB2’s Loopback0 subnet. But it’s also configured for LISP;

#### Lisp Configuration {#lispconfiguration}

<pre lang="shell">R2#sh run | sec router lisp

router lisp
 database-mapping 2001:10:1:22::/64 10.0.1.9 priority 1 weight 50
 database-mapping 2001:10:1:122::/64 10.0.1.9 priority 1 weight 50
!
 ipv6 alt-vrf LISP
 ipv6 itr map-resolver 10.0.0.13
 ipv6 itr
 ipv6 etr map-server 10.0.0.13 key LISP
 ipv6 etr</pre>

#### R2 Debugging {#r2debugging}

<pre lang="shell">R2#debug lisp control-plane all

1) *Apr 23 19:33:02.343: LISP: Processing data signal for EID prefix 2001:10:1:114::14/128
2) *Apr 23 19:33:02.347: LISP: Remote EID prefix 2001:10:1:114::14/128, Change state to incomplete (method: data-signal, state: unknown, rlocs: 0).
3) *Apr 23 19:33:02.355: LISP: Remote EID prefix 2001:10:1:114::14/128, Scheduling map requests (incomplete) (method: data-signal, state: incomplete, rlocs: 0).
4) *Apr 23 19:33:02.403: LISP: Send map request for EID prefix 2001:10:1:114::14/128
5) *Apr 23 19:33:02.403: LISP: Updating map request ITR RLOC array with 1 IPv4 entries (size 6).
6) *Apr 23 19:33:02.403: LISP: Remote EID prefix 2001:10:1:114::14/128, Send map request (1) (method: data-signal, state: incomplete, rlocs: 0).
7) *Apr 23 19:33:02.403: LISP: AF IPv6, Sending map-request from 2001:10:1:22::1 to 2001:10:1:114::14 for EID 2001:10:1:114::14/128, ITR-RLOCs 1, nonce 0xFFC9BD93-0x2757D544 (encap src 10.0.1.9, dst 10.0.0.13).</pre>

So **R2** begins the process of asking

  * Looks at the destination prefix
  * Notices that there’s no map-cache entry for it &#8211; Line 2
  * Builds a map-request entry for 2001:10:1:114::14 from it’s local connected subnet to RB2 in behalf or RB2 &#8211; Line 3
  * Sends the map-request packet for 2001:10:1::14/128 to 10.0.0.13 (Our MS_MR Router) &#8211; Line 7

Let’s take a look at single packet capture to see the UDP encapsulation of the LISP message and ports used. Just so you see I’m not making this stuff up, and neither is it’s draft standard ;).

<pre lang="shell">yandy@ymbp Lab Catures&gt; tshark -r R2Serial.pcap -V -R udp
Frame 22: 148 bytes on wire (1184 bits), 148 bytes captured (1184 bits)
    ……..
Point-to-Point Protocol
    ………
Internet Protocol, Src: 10.0.1.9 (10.0.1.9), Dst: 10.0.0.13 (10.0.0.13)
    Version: 4
    ……….
    Protocol: UDP (17)
    …….
    Source: 10.0.1.9 (10.0.1.9)
    Destination: 10.0.0.13 (10.0.0.13)
User Datagram Protocol, Src Port: lisp-control (4342), Dst Port: lisp-control (4342)
    Source port: lisp-control (4342)
    Destination port: lisp-control (4342)
    Length: 124
    ……….
Data (116 bytes)
……….</pre>

Some of the information has been removed, just to keep the output smaller. You can see right away source/destination of the UDP LISP message and it’s ports. UDP Port 4342 which is the lisp-control port as specified in it’s the [LISP Draft Standard][2].

Now, let’s take a look at our MS_MR router and see what it’s processing and doing with these requests.

#### Map Server / Map Resolver Debug Output {#mapservermapresolverdebugoutput}

<pre lang="shell">MS_MR#debug lisp control-plane all

1) *Apr 23 19:33:02.451: LISP: Processing received Encap-Control message from 10.0.1.9 to 10.0.0.13
2) *Apr 23 19:33:02.451: LISP: Processing received Map-Request message from 2001:10:1:22::1 to 2001:10:1:114::14
3) *Apr 23 19:33:02.451: LISP: Received map request, source_eid 2001:10:1:122::22, ITR-RLOCs: 10.0.1.9, records 1, nonce 0xFFC9BD93-0x2757D544
4) *Apr 23 19:33:02.451: LISP: MS registration prefix 2001:10:1:114::/64 10.0.1.1 site MainSite, Forwarding map request to ETR 10.0.1.1.</pre>

  * First thing we notice right way is the receipt of the Control Message from **R2** — Line 1
  * MR recognizes it’s a Map-Request message received from **R2** on behalf of **RB2** — Lines 2, 3
  * MR sees that this prefix is in MainSite and reachable via **C1**, so it forwards this request on to **C1**

Let’s see what happens when the Map Server / Responder forwards this request over to **C1**.

#### C1 Debugging {#c1debugging}

<pre lang="shell">C1#debug lisp control-plane all

1) *Apr 23 19:33:01.787: LISP: Processing received Encap-Control message from 10.0.1.33 to 10.0.1.1
2) *Apr 23 19:33:01.791: LISP: Processing received Map-Request message from 2001:10:1:22::1 to 2001:10:1:114::14
3) *Apr 23 19:33:01.791: LISP: Received map request, source_eid 2001:10:1:122::22, ITR-RLOCs: 10.0.1.9, records 1, nonce 0xFFC9BD93-0x2757D544</pre>

Once again we have pretty much the same information as we saw on **MS_MR** originally.

  * **C1** receives this forwarded map-request from **MS_MR** — line 1
  * **C1** Looks into the request packet to see what it’s trying to map 
      * This is a mapping to 2001:10:1:114::14 from Source EID 2001:10:1:122::22 — Lines 2, 3

Once **C1** examines it’s LISP database, which holds it’s local-site information as seen below.

#### C1’s LISP Configuration {#c1slispconfiguration}

<pre lang="shell">router lisp     
 database-mapping 2001:10:1:14::/64 10.0.1.1 priority 1 weight 50
 database-mapping 2001:10:1:114::/64 10.0.1.1 priority 1 weight 50
!
 ipv6 alt-vrf LISP
 ipv6 itr map-resolver 10.0.0.13
 ipv6 itr
 ipv6 etr map-server 10.0.0.13 key LISP
 ipv6 etr
!</pre>

<pre lang="shell">C1#sh ipv6 lisp database 
LISP ETR IPv6 Mapping Database, LSBs: 0x1, 2 entries

EID-prefix: 2001:10:1:14::/64
  10.0.1.1, priority: 1, weight: 50, state: site-self, reachable
EID-prefix: 2001:10:1:114::/64
  10.0.1.1, priority: 1, weight: 50, state: site-self, reachable</pre>

It then moves on to respond to **R2** directly on where 2001:10:1:114::14/128 is located.

#### C1 Debuggig {#c1debuggig}

<pre lang="shell">C1#debug lisp control-plane all

1) *Apr 23 19:33:01.791: LISP: Processing map request record for EID prefix 2001:10:1:114::14/128
2) *Apr 23 19:33:01.791: LISP: Sending map-reply from 10.0.1.1 to 10.0.1.9.
3) *Apr 23 19:33:01.791: LISP: Processing mapping information for EID prefix 2001:10:1:122::/64</pre>

  * **C1** sends map-reply from it’s 10.0.1.1 address to 10.0.1.9 (**R2’s**) address

**R2** upon receipt of this Map-Reply message starts to process the information and building it’s map-cache.

<pre lang="shell">R2#debug lisp control-plane all

1) *Apr 23 19:33:02.423: LISP: Processing received Map-Reply message from 10.0.1.1 to 10.0.1.9
2) *Apr 23 19:33:02.427: LISP: Received map reply nonce 0xFFC9BD93-0x2757D544, records 1
3) *Apr 23 19:33:02.427: LISP: Processing Map-Reply mapping record for 2001:10:1:114::/64, ttl 1440, state complete, authoritative, 1 locator
        10.0.1.1 pri/wei=1/50 LpR
4) *Apr 23 19:33:02.427: LISP: Map Request prefix 2001:10:1:114::14/128 remote EID prefix, Received reply with rtt 8ms.
5) *Apr 23 19:33:02.427: LISP: Processing mapping information for EID prefix 2001:10:1:114::/64
6) *Apr 23 19:33:02.427: LISP: Remote EID prefix 2001:10:1:114::/64, Change state to complete (method: map-reply, state: unknown, rlocs: 0).
7) *Apr 23 19:33:02.427: LISP: Remote EID prefix 2001:10:1:114::/64, Starting idle timer (method: map-reply, state: complete, rlocs: 0).
8) *Apr 23 19:33:02.427: LISP: Remote EID prefix 2001:10:1:114::14/128, Change state to deleted (method: data-signal, state: incomplete, rlocs: 0).
9) *Apr 23 19:33:02.431: LISP: Remote EID prefix 2001:10:1:114::/64 RLOC 10.0.1.1 pri/wei=1/50, Add/update locator flags LpR (method: map-reply, state: complete, rlocs: 0).</pre>

One of the things to note from the output above is that, mapping information is done for the complete subnet (Line 6) and not just **C4’s** individual address. **C1** responds with the mappings of its configured database, and it’s configured to map the entire /64 not individual hosts. Once this mapping is complete, the /128 is removed from the cache and the /64 added to the EID-to-RLOC map-cache (Lines 8 and 9).

# First Summary {#firstsummary}

So in summary, this is what happens for the ICMP Echo Request packet flowing from **RB2** to **C4**.

  * **RB2** sends ICMP packet towards it’s default gateway **R2**
  * **R2** tries to Locate **C4’s** prefix by requesting this information from **MS_MR**
  * **MS_MR** forwards this request to **C1**
  * **C1** replies directly to **R2** for the location of **C4’s** prefix
  * **R2** builds it’s map-cache table and includes this mapping
  * **R2** can now begin forwarding packets towards this destination

**Diagram of above explanation**

<img id="firstdiagram" title="" alt="First Diagram" src="http://pktmaniac.info/wordpress/images/LISP_RB2_Pings_C4.png" />

#### R2’s IPv6 Map-cache {#r2sipv6map-cache}

<pre lang="shell">R2#sh ipv6 lisp map-cache 
LISP IPv6 Mapping Cache, 2 entries

::/0, uptime: 00:00:12, expires: never, via static
  Negative cache entry, action: send-map-request
2001:10:1:114::/64, uptime: 00:00:08, expires: 23:59:44, via map-reply, complete
  Locator   Uptime    State      Pri/Wgt
  10.0.1.1  00:00:08  up           1/50</pre>

We can see that this prefix is now included in it’s IPv6 AFI map-cache and the locator for it is, 10.0.1.1 (**C1**).

# Second Ping Request {#secondpingrequest}

So, now that **R2** knows how to reach **C4** at 2001:10:1:114::14; does that mean everything works? Well if you consider one-way communication a success, then yes, technically it works, and **RB2** can reach **C4**. However, at this point return traffic is not able to make it back to **RB2**, so Echo-Replies can’t be seen yet.

So, let’s see what happens when **C4** tries to reply back to **RB2**, which essentially is the same process.

  * **C1** consults its routing and LISP map-caches
  * Having no cache entry for 2001:10:1:122::22/128
  * **C1** builds and sends map-request towards the **MS**

<pre lang="shell">1) *Apr 23 19:33:03.743: LISP: Processing data signal for EID prefix 2001:10:1:122::22/128
2) *Apr 23 19:33:03.747: LISP: Remote EID prefix 2001:10:1:122::22/128, Change state to incomplete (method: data-signal, state: unknown, rlocs: 0).
2) *Apr 23 19:33:03.747: LISP: Remote EID prefix 2001:10:1:122::22/128, Scheduling map requests (incomplete) (method: data-signal, state: incomplete, rlocs: 0).
3) *Apr 23 19:33:03.759: LISP: Send map request for EID prefix 2001:10:1:122::22/128
4) *Apr 23 19:33:03.759: LISP: Updating map request ITR RLOC array with 1 IPv4 entries (size 6).
5) *Apr 23 19:33:03.759: LISP: Remote EID prefix 2001:10:1:122::22/128, Send map request (1) (method: data-signal, state: incomplete, rlocs: 0).
6) *Apr 23 19:33:03.759: LISP: AF IPv6, Sending map-request from 2001:10:1:14::1 to 2001:10:1:122::22 for EID 2001:10:1:122::22/128, ITR-RLOCs 1, nonce 0x4BFE7B89-0x995FDD9B (encap src 10.0.1.1, dst 10.0.0.13).</pre>

  * Once the **MS** receives this map-request packet, if consults it’s *Site Table*

<pre lang="shell">MS_MR#sh lisp site 2001:10:1:122::/64 
LISP Site Registration Information

Site name: SiteTwo
Description: R2_RB2
Allowed configured locators: any
Requested EID-prefix:
  EID-prefix: 2001:10:1:122::/64
    First registered:     1d03h
    Routing table tag:    0x0
    Origin:               Configuration
    Registration errors:  
      Authentication failures:   0
      Allowed locators mismatch: 0
    ETR 10.0.1.9, last registered 00:00:16, no proxy-reply, no map-notify
                  TTL 1d00h
      Locator   Local  State      Pri/Wgt
      10.0.1.9  yes    up           1/50</pre>

  * Having found 10.0.1.9 **R2** as the locator for this prefix (2001:10:1:122::22/128
  * The **MS** forwards the request on to the proper ETR, **R2** in this case

<pre lang="shell">1) *Apr 23 19:33:04.419: LISP: Processing received Encap-Control message from 10.0.1.1 to 10.0.0.13
2) *Apr 23 19:33:04.427: LISP: Processing received Map-Request message from 2001:10:1:14::1 to 2001:10:1:122::22
3) *Apr 23 19:33:04.431: LISP: Received map request, source_eid 2001:10:1:114::14, ITR-RLOCs: 10.0.1.1, records 1, nonce 0x4BFE7B89-0x995FDD9B
4) *Apr 23 19:33:04.431: LISP: MS registration prefix 2001:10:1:122::/64 10.0.1.9 site SiteTwo, Forwarding map request to ETR 10.0.1.9.</pre>

  * Upon receipt of this map-request form **C1** , **R2** looks up it’s LISP IPv6 database

<pre lang="shell">R2#sh ipv6 lisp dat
LISP ETR IPv6 Mapping Database, LSBs: 0x1, 2 entries

EID-prefix: 2001:10:1:22::/64
  10.0.1.9, priority: 1, weight: 50, state: site-self, reachable
EID-prefix: 2001:10:1:122::/64
  10.0.1.9, priority: 1, weight: 50, state: site-self, reachable</pre>

  * Being the xTR for this specific prefix, **R2** builds a map-reply message back to **C1**

<pre lang="shell">1) *Apr 23 19:33:04.399: LISP: Processing received Encap-Control message from 10.0.1.33 to 10.0.1.9
2) *Apr 23 19:33:04.399: LISP: Processing received Map-Request message from 2001:10:1:14::1 to 2001:10:1:122::22
3) *Apr 23 19:33:04.399: LISP: Received map request, source_eid 2001:10:1:114::14, ITR-RLOCs: 10.0.1.1, records 1, nonce 0x4BFE7B89-0x995FDD9B
4) *Apr 23 19:33:04.399: LISP: Processing map request record for EID prefix 2001:10:1:122::22/128
5) *Apr 23 19:33:04.399: LISP: Sending map-reply from 10.0.1.9 to 10.0.1.1.</pre>

This message basically says; if you need to reach 2001:10:1:122::22/128 add the 2001:10:1:122::/64 to your map-cache. Send all packets to 10.0.1.9 that are destined to the 2001:10:1:122::/64 subnet.

**C1** gets a hold of this reply message and begins adding it to it’s local dynamic map-cache table. And it’s only at this point that we can have two-way communication between the two sites.

<pre lang="shell">1) *Apr 23 19:33:03.779: LISP: Processing received Map-Reply message from 10.0.1.9 to 10.0.1.1
2) *Apr 23 19:33:03.783: LISP: Received map reply nonce 0x4BFE7B89-0x995FDD9B, records 1
3) *Apr 23 19:33:03.783: LISP: Processing Map-Reply mapping record for 2001:10:1:122::/64, ttl 1440, state complete, authoritative, 1 locator 10.0.1.9 pri/wei=1/50 LpR
4) *Apr 23 19:33:03.783: LISP: Map Request prefix 2001:10:1:122::22/128 remote EID prefix, Received reply with rtt 12ms.
5) *Apr 23 19:33:03.783: LISP: Processing mapping information for EID prefix 2001:10:1:122::/64
6) *Apr 23 19:33:03.783: LISP: Remote EID prefix 2001:10:1:122::/64, Change state to complete (method: map-reply, state: unknown, rlocs: 0).
7) *Apr 23 19:33:03.783: LISP: Remote EID prefix 2001:10:1:122::/64, Starting idle timer (method: map-reply, state: complete, rlocs: 0).
8) *Apr 23 19:33:03.783: LISP: Remote EID prefix 2001:10:1:122::22/128, Change state to deleted (method: data-signal, state: incomplete, rlocs: 0).
9) *Apr 23 19:33:03.783: LISP: Remote EID prefix 2001:10:1:122::/64 RLOC 10.0.1.9 pri/wei=1/50 Add/update locator flags LpR (method: map-reply, state: complete, rlocs: 0).</pre>

It seems like a pretty lengthly process, specially going by how big this post has gotten. However, all these things happen at a pretty fast pace in a router. I would like to say **wire-rate** but that’s not always the case with all features and or hardware.

Here’s **C1’s** map-cache entry for **RB2’s** subnet.

<pre lang="shell">C1#sh ipv6 lisp map-cache
LISP IPv6 Mapping Cache, 2 entries

::/0, uptime: 00:00:34, expires: never, via static
  Negative cache entry, action: send-map-request
2001:10:1:122::/64, uptime: 00:00:22, expires: 23:59:30, via map-reply, complete
  Locator   Uptime    State      Pri/Wgt
  10.0.1.9  00:00:22  up           1/50</pre>

Now that both tables are filled, and two way communication can resume, at this point is where the “!!!” start to appear in our ping output.

Here’s a diagram of the process described (C4 replying to RB2)

<img id="diagram2" title="" alt="Diagram 2" src="http://pktmaniac.info/wordpress/images/LISP_C4_Replies_RB2.png" />

  * Just for fun, here’s a single packet capture of the LISP-data being sent back and forth.

<pre lang="shell">Frame 21: 140 bytes on wire (1120 bits), 140 bytes captured (1120 bits)
    ……..
Point-to-Point Protocol
    …….
Internet Protocol, Src: 10.0.1.9 (10.0.1.9), Dst: 10.0.1.1 (10.0.1.1)
    Version: 4
    ……
    Time to live: 255
    Protocol: UDP (17)
    ……..
    Source: 10.0.1.9 (10.0.1.9)
    Destination: 10.0.1.1 (10.0.1.1)
User Datagram Protocol, Src Port: 1024 (1024), Dst Port: lisp-data (4341)
    Source port: 1024 (1024)
    Destination port: lisp-data (4341)
    Length: 116
……
Data (108 bytes)

…….</pre>

Couple things to note are the source and destination addresses, even though it’s an IPv6 ICMP Echo-request. This packet is sent in an IPv4 packet, using UDP as it’s transport and destination port of (4341) lisp-data. Also, the TTL is set to 255, which hides the path of the packet as it traverses the network. Kind of how MPLS hides traceroutes as well when configured to not propagate it’s TTL. Which means if you trace from **RB2** towards **C4** or any direction, and this needs to be LISP tunneled, then you’ll only see end-to-end hops.

<pre lang="shell">RB2#trace 2001:10:1:114::14
Type escape sequence to abort.
Tracing the route to 2001:10:1:114::14

  1 2001:10:1:22::1 12 msec 0 msec 4 msec
  2 2001:10:1:14::1 16 msec 12 msec 12 msec
  3 2001:10:1:14::14 12 msec 16 msec 16 msec</pre>

Our IPv4 network is hidden from the traces, and only the LISP aware sites are shown. If you download and look at the included Capture files in <a href="http://www.wireshark.org" target="blank">Wireshark</a> you’ll also notice that once the packets reach / or before they get LISP tunneled, they’re just normal ICMPv6 packets.

# Summary {#summary}

#### Lisp the Good {#lispthegood}

  * Fairly easy to pick up and learn, if proper time is taken
  * Seems flexible, extendable and useful
  * IPv4 and IPv6 agnostic, doesn’t care what your run, or what over
  * Fairly “fast”’ish in converging, although there’s allot of control information in the background
  * While, not the fastest to converge, it is stable once it’s finished

#### Improvements to LISP? {#improvementstolisp}

Improvements to the protocol itself are hard to say at this point. Needs a bit more testing, I do wish it was a bit faster, maybe a type of pre-caching before any actual data needs to pass. This may in fact remove what the protocol was meant to solve, which is Routing Table schemes and Sizes, so it may not be feasible.  
At least on the Cisco configuration, I can’t find any information on changing map-cache stale time. Or Map Server / Resolver priority, seems to pick highest-IP type of thing, or sometimes flooding requests to all configured MS/MR. This is a topic for another post though.

#### Final Thoughts {#finalthoughts}

Finally, the good part, the end of this long a** post right? ;). Hopefully it’s been somewhat useful and or informative and not mis-leading. As of now, I like LISP and I’m interested in it’s development, it seems like it may have a place after all in our networks. For which purpose it’s used, really depends on needs, like just about everything we engineer / design in this industry.

 [1]: {{ site.url }}/assets/images/LISP_IPv6_IPv4.png
 [2]: https://datatracker.ietf.org/doc/draft-ietf-lisp/?include_text=1