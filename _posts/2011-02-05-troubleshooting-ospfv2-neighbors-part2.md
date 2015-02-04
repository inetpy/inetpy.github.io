---
title: Troubleshooting OSPFv2 Neighbors (Part2)
author: yandy
layout: post
permalink: /2011/02/troubleshooting-ospfv2-neighbors-part2/
jabber_published:
  - 1296928848
dsq_thread_id:
  - 2479849711
categories:
  - Routing
tags:
  - ccie
  - neighbors
  - Troubleshooting
  - routing
  - ospf
  - ospf neighbors
  - cisco
---
This will seem very similar to t he last post, again going on simple. Things that can be overlooked if one is not careful, doesn't look at everything with a magnifying glass, and becomes over confident. Same scenario, routers, link and the same area as before, but again just a little bit of enough twist to have an issue.

The debugging commands pertain to OSPFv2, however, the same thing can happen with any other IGP that requires peering. I'll cover two different ways of really configuring the same mistake, or overlooking the same mistake by someone else.

**Diagram**

[<img class="alignnone" src="http://ipyandy.net/assets/images/r8-r4.png" alt="" width="231" height="243" />][1]

&nbsp;

**<!--more-->Getting to the issue**

Checking neighbors on both routers reveals some time of problem, that ospf adjacencies are not forming. So you go through your usual steps; (I have logging buffered only)

{% highlight bash %}
r8#sh ip ospf neigh
r8#

r4#sh ip ospf neigh
r4#
{% endhighlight %}

Having seen the neighbor table empty, let's say we decide to debug the hello packets and see if anything shows up.

{% highlight bash %}
r8#debug ip ospf hello  
OSPF hello events debugging is on  
r8#

r8#sh logg……. &#8212; output removed  
*Mar  7 23:56:53.254: OSPF: Send hello to 224.0.0.5 area 48 on FastEthernet0/0 from 172.8.48.8 
Mar  7 23:57:03.170: OSPF: Send hello to 224.0.0.5 area 48 on FastEthernet0/0 from 172.8.48.8</p> 

r4#debug ip ospf hello  
OSPF hello events debugging is on  
r4#

r4#sh logg&#8230;.. OUTPUT REMOVED  
*Mar  8 00:00:51.990: OSPF: Send hello to 224.0.0.5 area 48 on FastEthernet0/1 from 172.8.40.4  
*Mar  8 00:01:01.266: OSPF: Send hello to 224.0.0.5 area 48 on FastEthernet0/1 from 172.8.40.4  
*Mar  8 00:01:10.634: OSPF: Send hello to 224.0.0.5 area 48 on FastEthernet0/1 from 172.8.40.4  
*Mar  8 00:01:19.918: OSPF: Send hello to 224.0.0.5 area 48 on FastEthernet0/1 from 172.8.40.4
{% endhighlight %}

So we are sending hellos out the interfaces, however we don't seem to be seeing anything back; both routers send, yet they don't see each others hellos. Or do they? Since it's a different problem, we have to approach it a bit differently.

Let's debug the adjacency packets and see what happens;

{% highlight bash %}
r8#debug ip ospf adj   
OSPF adjacency events debugging is on  
r8#

r8#sh logg&#8230;&#8230;&#8230;&#8230;&#8230;..
Mar  8 00:04:35.266: OSPF: Rcv pkt from 172.8.40.4, FastEthernet0/0, area 0.0.0.48 : src not on the same network</strong>  
Mar  8 00:04:44.598: OSPF: Rcv pkt from 172.8.40.4, FastEthernet0/0, area 0.0.0.48 : src not on the same network
r8#
{% endhighlight %}

If we look a little closer at R8's output, we notice something right away, src not on same subnet. Let's take a look at the configuration and see what we have:

**R8**
{% highlight bash %}
r8#sh run int f0/0  
Building configuration&#8230;  
Current configuration : 113 bytes!  
interface FastEthernet0/0   
 ip address 172.8.48.8 255.255.255.0   
 ip ospf 1 area 48   
 speed 100   
full-duplex  
end

R4

r4#sh run int f0/1  
Building configuration&#8230;  
Current configuration : 133 bytes!  
interface FastEthernet0/1  
ip address 172.8.40.4 255.255.255.0  
ip ospf priority 0  
ip ospf 1 area 48  
speed 100  
full-duplex  
end
{% endhighlight %}

Going by the output of the debug, and looking at the interface config, we can see right away that R4's interface has a different subnet configured than R8. one is in the 172.8.48 subnet and the other in the 172.8.40. subnet. Looking at it here is a bit easier to spot. Sometimes, when you're looking at a command line, that 0 (zero) and the 8 (eight) are very similar depending on font.

All we need to do is change R4 or R8, but we'll change R4 because the diagram says is the 172.8.48. subnet. Once that's changed, adjacencies will form and all will be normal. This was somewhat similar as the mismatched SUBNET MASK post, but two different ways to troubleshoot. The same command that worked on one, did not in the other.

We want to have as may weapons in the arsenal as possible, and know them right away. Wether in the lab or in a production environment.

The second part of this is really simple:

We can have the same problem, but have different output all together, if we were using the *network 172.8.48.0* command under OSPF process on R4, and not the *ip ospf* command under the interface like we are here. The result would be completely different, R4 would not even send hellos out, and the issue would become a bit more apparent.

 [1]: http://ipyandy.net/assets/images/r8-r4.png