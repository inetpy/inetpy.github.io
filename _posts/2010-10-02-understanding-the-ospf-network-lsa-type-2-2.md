---
title: Understanding the OSPF Network LSA (Type 2)
author: yandy
layout: post
permalink: /2010/10/understanding-the-ospf-network-lsa-type-2-2
image: generic_banner.jpg
twitter_image: generic_banner.jpg
description: 
summary: In this second part of our OSPF LSAs covering the Network LSA or otherwise known as Type 2 LSA. This is probably one of the easiest to understand. But there's no better way to tell, than to actually see it do it's thing...
dsq_thread_id:
  - 2479834179
categories:
  - Networking
tags:
  - ccie
  - cisco-ospf
  - network-lsa
  - ospf
  - ospf-lsa
  - routing
---
<hr>
![](http://ipyandy.net/images/generic_banner.jpg)
<hr>

In this second part of our OSPF LSAs covering the Network LSA or otherwise known as Type 2 LSA. This is probably one of the easiest to understand. But there's no better way to tell, than to actually see it do it's thing.
  
### Description

The OSPF Network LSA (type 2) is advertised by the DR (Designated Router) on each multi-access network. The DR itself represents the multi-access network itself and all of it's attached routers (quote from Routing TCP/IP Vol1) as a pseudonode. This means that the DR itself has a &#8220;virtual&#8221; adjacency to the pseudonode as the Network LSA would show. The Network LSA represents this &#8220;virtual router&#8221; just like the Router LSA (type 1) represented a router itself as discussed in the <a href="http://ipyandy.net/2011/05/the-ospfv3-router-lsa/" target="_blank">previous</a>post. The Network LSA lists all attached routers to that multi-access network. Again, this is only advertised by the DR and it has area local scope.
  
One thing to note is the lack of a metric in the Network LSA. This is because the cost from this pseudonode (virtual router) to any attached router is always zero. This prevents the pseudonode from adding extra cost to the link as it is only a virtual instance representing a multi-access network.

 What does this all mean? Well lets take a look at some examples to better explain this. Below is the topology we'll be using, it'll be the same for all current and future OSPF LSA posts, unless specified.

#### Topology
<a href="{{ site.url }}/assets/images/ospf-001-w_rip.png"><img style="border:0 initial initial;" src="{{ site.url }}/assets/images/ospf-001-w_rip.png" alt="" width="670" height="552" />

<!--more-->
Let's get started with a show command from R5, in this network R1 is the DR and R2 is the BDR, R5 would be a DROTHER to both R1 and R2.
  
<a href="{{ site.url }}/assets/images/subnet-125-segment.png"><img style="border:0 initial initial;" src="{{ site.url }}/assets/images/subnet-125-segment.png" alt="" width="" height="" />
  
R5's view of a Network LSA in the multi-access network 10.1.125.0/24</

<a href="{{ site.url }}/assets/images/network-lsa-r5.png"><img style="display:block;margin-left:auto;margin-right:auto;border:0 initial initial;" src="{{ site.url }}/assets/images/network-lsa-r5.png" alt="" width="673" height="199" />
  
 If you take a look and notice, this output is not truncated, there's 3 routers on the network but there's only one Network LSA. Because R5 is not the DR and R2 is only a BDR they do not produce a Network (type 2) LSA. As you can see from the output, this LSA has all routers attached on its multi-access network including R1, the DR itself. The routers are hard coded with a router-id in the OSPF process, of the router number in each octet (which is why they show as 1.1.1.1 and 2.2.2.2 and so on).

Noticing that even R1 is included in this LSA, takes you to the the statement of the DR being represented by a virtual instance of a router. This pseudonode has an adjacency to all attached routers on the network, including the DR. The lack of a metric is explained earlier and it's always = zero.

Let's take a look at the DRs (R1) view of this multi-access network through the Network LSA.

<a href="{{ site.url }}/assets/images/network-lsa-r1.png"><img style="display:block;margin-left:auto;margin-right:auto;border:0 initial initial;" src="{{ site.url }}/assets/images/network-lsa-r1.png" alt="" width="670" height="197" />
  
If you notice most of the information seen, is exactly the same. Hence it's the same LSA received by all routers on this network. The only difference is, the Router ID (in the second line) because we're viewing R1s database and this is the router that produced this LSA. Also the LS age is older, since I took some time to create this output after going through R5.

Once again, the same routers are described, those being R1, R2 and R5. This pseudonode or virtual router is representing the ethernet network and has a virtual adjacency to all attached routers, even R1. R1 is the DR and this pseudonode has adjacencies in the network through R1 as the DR.

Next OSPF LSA post, I'll cover the Network Summary LSA (type 3) and it's functions in the OSPF network. Once again, feel free to leave in any comments or rants.