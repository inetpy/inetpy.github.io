---
title: Troubleshooting OSPFv2 Neighbors (Part 1)
author: yandy
layout: post
permalink: /2011/01/troubleshooting-ospfv2-neighbors-part-1
image: generic_banner.jpg
twitter_image: generic_banner.jpg
description: 
summary: Tackling one of the simplest OSPFv2 adjacency problems to the trained eye. Yet, it's really incredible how often it can escape even the most seasoned veteran. Getting right to the point, let's begin with the following scenario below...
jabber_published:
  - 1296356918
dsq_thread_id:
  - 2479847343
categories:
  - Routing
tags:
  - ccie
  - troubleshooting
  - cisco
  - ospf
  - ospf-neighbors
  - neighbors
  - routing
---
<hr>
![](http://inetpy.com/images/generic_banner.jpg)
<hr>

Tackling one of the simplest OSPFv2 adjacency problems to the trained eye. Yet, it's really incredible how often it can escape even the most seasoned veteran. Getting right to the point, let's begin with the following scenario below:

[<img id="img" title="img" alt="img" src="http://inetpy.com/images/r8-r4.png" width="NaN" height="NaN" />][img1]

Here's a simple two router setup on an ethernet segment. They're trying to form a neighbor relationship in area 48. Here is the show run of the ethernet interfaces for both routers.

<!--more-->

[<img id="img" title="img" alt="img" src="http://inetpy.com/images/r4-run-int.png" width="NaN" height="NaN" />][img2]

If taking just a quick glance, the obvious may elude you as simple as it is; and it happens more often than not. Taking a look at both routers neighbor database, you'll notice that it's empty. By the way, those are the only two interfaces participating in OSPF at the moment.

[<img id="img" title="img" alt="img" src="http://inetpy.com/images/both-show-ospf-neigh.png" width="NaN" height="NaN" />][img3]

Let's do a quick debug of the hellos to make sure that they're at least seeing each other in their respective hellos.

**Here's R4s debugging of HELLOS**[  
[<img id="img" title="img" alt="img" src="http://inetpy.com/images/r4-debug-ospf-hello.png" width="NaN" height="NaN" />][img4]

**Here's R8s debugging of HELLOS** 
[<img id="img" title="img" alt="img" src="http://inetpy.com/images/r8-debug-ospf-hello.png" width="NaN" height="NaN" />][img5]

Clear to see here that both R4 and R8's hellos are being seen by each other, but if you look closely, something sticks out..

look at the line **Mismatched hello parameters from 172.8.48.8 **on R4, which would be the same on R8 but with a different address.

looking right below that, we see that the **dead timers** are the same **40 each** (this must match) the **Hello timers** are also the same **10 each** (these must also match) and they do.  
then you look at the **mask**, the (**R**)eceived mask is **255.255.255.0** and the (**C**)onnected mask is **255.255.254.0**

In OSPF the mask advertised by routers becoming neighbors must match for a giving interface. There are some exceptions, such as some point-2-point links and unnumbered interfaces..  A simple fix of R4s F0/1 interface mask and all is well:

[<img id="img" title="img" alt="img" src="http://inetpy.com/images/r4-neighbors-up.png" width="NaN" height="NaN" />][img6]

now we notice that R4 has R8 as it's neighbor, and R8 is the DR for the segment.

Simple, yet somehow these simple things are the ones that escape us the most. As engineers, sometimes we tend to overcomplicate allot of things, that are really not necessary.

[img1]: http://inetpy.com/images/r8-r4.png
[img2]: http://inetpy.com/images/r4-run-int.png
[img3]: http://inetpy.com/images/both-show-ospf-neigh.png
[img4]: http://inetpy.com/images/r4-debug-ospf-hello.png
[img5]: http://inetpy.com/images/r8-debug-ospf-hello.png
[img6]: http://inetpy.com/images/r4-neighbors-up.png