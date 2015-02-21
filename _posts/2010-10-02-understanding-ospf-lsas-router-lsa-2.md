---
title: Understanding OSPF LSAs (Router LSA)
author: yandy
layout: post
permalink: 2010/10/understanding-ospf-lsas-router-lsa-2/
image: generic_banner.jpg
twitter_image: generic_banner.jpg
description: 
summary: This first post is dedicated to the first of the OSPF LSAs which is. Router (type 1),  I've met / interviewed people that know exactly what most OSPF LSA types are by the book. But present a small scenario and ask what type of LSA will be generated or seen and they stutter or stare into blank space. Hence it's not a true understanding of what these are, and I'll try to cover that in this post...
dsq_thread_id:
  - 2479863838
categories:
  - Networking
  - OSPF
tags:
  - ccie
  - cisco
  - cisco-networking
  - ospf-lsa
  - router-lsa
  - routing
  - ospf
  - cisco-ospf
  - lsa-type-2
---
<hr>
![](http://ipyandy.net/images/generic_banner.jpg)
<hr>

This first post is dedicated to the first of the OSPF LSAs which is. Router (type 1),  I've met / interviewed people that know exactly what most OSPF LSA types are by the book. But present a small scenario and ask what type of LSA will be generated or seen and they stutter or stare into blank space. Hence it's not a true understanding of what these are, and I'll try to cover that in this post.
  
Future posts will cover Network LSA (type 2), Network Summary LSA (type 3), ASBR summary LSA (type 4), Autonomous System External LSAs or just plain External LSAs (type 5), and NSSA External LSAs (type 7). But for now, lets just focus on the first one of these before going any deeper.

<!--more-->
  
Here's the diagram for the topology, nothing fancy, the frame cloud is setup as ospf network type point-to-multipoint to make it simple. The DR and BDR for each segment are predictable, R1 and R2 are always the DR and or BDR.
  
#### OSPF Topology ####
<img id="ospf-001" title="ospf-001-w_rip" alt="ospf-001-w_rip" src="{{ site.url }}/assets/images/ospf-001-w_rip.png" />
  
{% highlight bash %}
r1#sh ip ospf database router

Router Link States (Area 0)
OSPF Router with ID (1.1.1.1) (Process ID 1)
Router Link States (Area 0)
  
LS age: 1899
Options: (No TOS-capability, DC)
LS Type: Router Links
Link State ID: 1.1.1.1
Advertising Router: 1.1.1.1
LS Seq Number: 8000000B
Checksum: 0x7745
Length: 72
Area Border Router
Number of Links: 4

Link connected to: another Router (point-to-point)
Link ID) Neighboring Router ID: 3.3.3.3
Link Data) Router Interface address: 10.1.123.1
Number of TOS metrics: 0
TOS 0 Metrics: 64

Link connected to: another Router (point-to-point)
Link ID) Neighboring Router ID: 2.2.2.2
Link Data) Router Interface address: 10.1.123.1
Number of TOS metrics: 0
TOS 0 Metrics: 64

Link connected to: a Stub Network
Link ID) Network/subnet number: 10.1.123.1
Link Data) Network Mask: 255.255.255.255
Number of TOS metrics: 0
TOS 0 Metrics: 0

Link connected to: a Stub Network
Link ID) Network/subnet number: 10.0.0.1
Link Data) Network Mask: 255.255.255.255
Number of TOS metrics: 0
TOS 0 Metrics: 1
Routing Bit Set on this LSA
LS age: 54
Options: (No TOS-capability, DC)
LS Type: Router Links
Link State ID: 2.2.2.2
Advertising Router: 2.2.2.2
LS Seq Number: 8000000B
Checksum: 0x5163
Length: 72
Area Border Router
Number of Links: 4

Link connected to: another Router (point-to-point)
Link ID) Neighboring Router ID: 3.3.3.3
Link Data) Router Interface address: 10.1.123.2
Number of TOS metrics: 0
TOS 0 Metrics: 64

Link connected to: another Router (point-to-point)
Link ID) Neighboring Router ID: 1.1.1.1
Link Data) Router Interface address: 10.1.123.2
Number of TOS metrics: 0
TOS 0 Metrics: 64

Link connected to: a Stub Network
Link ID) Network/subnet number: 10.1.123.2
Link Data) Network Mask: 255.255.255.255
Number of TOS metrics: 0
TOS 0 Metrics: 0
Link connected to: a Stub Network
(Link ID) Network/subnet number: 10.0.0.2
(Link Data) Network Mask: 255.255.255.255
Number of TOS metrics: 0
TOS 0 Metrics: 1
{% endhighlight %}
  
 I didn't include all the router LSAs received, just enough for relevance. Lets examine R1's Router LSAs for Area 0.

Notice how LSA with ID 1.1.1.1 (R1) includes two neighboring routers (2.2.2.2 and 3.3.3.3 R2 and R3 respectively).

Also includes links to Stub Networks; but what does that mean? Well on of those, (Link ID 10.0.0.1) is a loopback address on R1, loopback addresses by default are advertised as stub networks or /32 routes. That loopback happens to be configured with a /32 but doesn't matter the netmask the outcome is the same, go ahead try it.

The other stub network (Link ID 10.1.123.1) is advertised as /32 as well due to the configuration on interface s0/0, which is set to ospf point-to-multipoint network (if anyone wants to see a post on network/interface types let me know).

Key thing to notice here is, that R1 is advertising in a Router LSA all it's directly connected links, and the neighbors on those links.

Now let's take a look at Area's 125 LSA information from the perspective of R5.
  
{% highlight bash %}
r5#sh ip ospf database router

Router Link States (Area 125)
Routing Bit Set on this LSA
LS age: 1320
Options: (No TOS-capability, DC)
LS Type: Router Links
Link State ID: 1.1.1.1
Advertising Router: 1.1.1.1
LS Seq Number: 80000009
Checksum: 0xDE27
Length: 36
Area Border Router
Number of Links: 1
Link connected to: a Transit Network
Link ID) Designated Router address: 10.1.125.1
Link Data) Router Interface address: 10.1.125.1
Number of TOS metrics: 0
TOS 0 Metrics: 10
{% endhighlight %}

See how R1 advertises itself as the DR for that multi-access network, the DR address and the interface address are the same.
  
{% highlight bash %}
LS age: 1119
Options: (No TOS-capability, DC)
LS Type: Router Links
Link State ID: 5.5.5.5
Advertising Router: 5.5.5.5
LS Seq Number: 8000000A
Checksum: 0x9B26
Length: 48
Number of Links: 2

Link connected to: a Transit Network
Link ID) Designated Router address: 10.1.125.1
Link Data) Router Interface address: 10.1.125.5
Number of TOS metrics: 0
TOS 0 Metrics: 10
{% endhighlight %}
  
R5 sees R1 as the DR, currently R5 is configured to never become the DR for that network. And it advertises it's IP into that area from F0/0.125.

{% highlight bash %}
Link connected to: a Stub Network
Link ID) Network/subnet number: 10.0.0.5
Link Data) Network Mask: 255.255.255.255
Number of TOS metrics: 0
TOS 0 Metrics: 1
{% endhighlight %}

No DR information needed for a loopback address, as it does not connect to any routers, again notice how it's netmask is a /32 (255.255.255.255) since it's a stub network. This can be changed with the command *ip ospf network point-to-point* under the loopback interface.

Again, really much of the same information is included (I again, left out output, like R2s router lsa into that area).  Looking at *link state id 1.1.1.1 (R1)*, you see advertising it's only directly connected link/interface into that area (you'll notice *Number of Links set to 1)* and *Link state ID 5.5.5.5 (R5)* number of links at 2. Since all of R5s router interfaces are in area 125.

The main thing to take from this is, that a Router LSA (LSA Type 1) is generated by all routers, within and area, for all their areas. Only the links (interfaces) connected to that area are advertised in the Router LSA for that Area. This is by far probably the simplest LSA type to understand.
  
 I was going to include three LSA types per blog post, but the post as you can see can get out of hand in length. I decided to break each LSA up into it's own post and next one will contain OSPFs Network LSA (type 2).

 Feedback and or questions are always welcome