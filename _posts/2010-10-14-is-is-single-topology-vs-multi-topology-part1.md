---
title: IS-IS Single-Topology vs Multi-Topology
author: yandy
layout: post
permalink: /2010/10/is-is-single-topology-vs-multi-topology-part1
dsq_thread_id:
  - 2479854059
categories:
  - Networking
tags:
  - arista
  - cisco
  - ipv6
  - isis
  - multi-topology
  - networking
  - routing
  - single-topoly
---
Today I'm covering a little topic that seems to trouble some people. Mainly because IS-IS is really only used in provider environments, but I happen to like it. It's a fairly stable protocol and once you get passed the OSPF vs IS-IS fight, it just works. Personally, I'll choose OSPF over IS-IS in most occasions just because more people know OSPF, but that's it. Either way, today's topic is for people running both IPv6 and IPv4 in the same network (routers / core). I've actually seen one environment where their IPv4 and IPv6 environment were completely separate.

**What is Single-topology IS-IS**? It's not what the name may imply, does not mean you're running only one protocol (IS-IS for either IPv4 or IPv6). What it does mean is that, every interface running IS-IS must be configured with an identical set of address-families. And all routers in the routing domain, must support the same address-families as well. This does not mean that an interface has to have both an IPv4 and IPv6 address, but it must be able to support it, as soon as you type **ipv6 unicast-routing** most modern routers will enable the address family on all interfaces.

The characteristics of all interfaces running IS-IS are shared, such as metrics and metric types. I always run **metric-style wide** for all interfaces running the protocol although narrow metrics are supported if IPv6 is used, wide metrics is advertised in the TLVs for IPv6. If you don't know what that is, I suggest reading up on IS-IS basics as this is not meant to be a primer to it. An interface running both IPv4 and IPv6 IS-IS will share it's metric, even if configured differently.

Here's the topology we'll be looking at today. 
<a href="{{ site.url }}/assets/images/is-is.png"><img class="aligncenter" title="IS-IS" alt="" src="{{ site.url }}/assets/images/is-is.png" width="679" height="511" /></a>

<!--more-->

Let's concentrate on the serial link between R2 and R8 (S1/3 on each).

**R2**
<a href="{{ site.url }}/assets/images/r2-s1-3.png"><img class="alignnone" title="r2" alt="" src="{{ site.url }}/assets/images/r2-s1-3.png" width="444" height="155" /></a>

**R8**
<a href="{{ site.url }}/assets/images/r8-s1-3.png"><img class="alignnone" title="r8" alt="" src="{{ site.url }}/assets/images/r8-s1-3.png" width="442" height="144" /></a>

You can see that both R2 and R8 are configured for a metric of 20 for IPv4, and a metric of 3 for IPv6, on their respective serial interfaces. However looking at the IS-IS routing table for both address-families reveals something interesting, (will only show one router) as the outcome is the same on both.
  </p>
  
**R8 IS-IS Config**
<a href="{{ site.url }}/assets/images/r8-isis-config.png"><img class="alignnone" title="r8 isis config" alt="" src="{{ site.url }}/assets/images/r8-isis-config.png" width="296" height="50" /></a>

**R8 IPv4 Routing Table (IS-IS)**
<a href="{{ site.url }}/assets/images/r8-ipv4-table.png"><img class="alignnone" title="R8-ipv4-table" alt="" src="{{ site.url }}/assets/images/r8-ipv4-table.png" width="403" height="74" /></a>
  
**R8 IPv6 Routing Table (IS-IS)**
<a href="{{ site.url }}/assets/images/r8-ipv6-table.png"><img class="alignnone" title="r8-ipv6-table-isis" alt="" src="{{ site.url }}/assets/images/r8-ipv6-table.png" width="463" height="141" /></a>

comparing the tables to the configuration above, you can see that although both protocols are configured to have different metrics. They actually share the same metric value through both L3 address-families.
  
**EDIT**
  
Basically in order to stop the sharing of metrics and use individual calculations for the address families. The configuration is very simple and straight forward.
  {% highlight bash %}
router isis
    metric-style wide
    address-family ipv6
      exit-address-family
{% endhighlight %}

You must enable wide metrics and configure multi-topolgy under IPv6 address family. The  **transition** part is optional and only used up until the point where all links are IPv6 ready.